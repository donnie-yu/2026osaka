<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OSAKA 2026 協作行程</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=SF+Pro+Display:wght@400;700;900&display=swap');
        body { font-family: 'SF Pro Display', -apple-system, sans-serif; background-color: #C2DFFF; margin: 0; }
       .no-scrollbar::-webkit-scrollbar { display: none; }
        #itinerary-content.pdf-mode { width: 210mm; min-height: 297mm; padding: 10mm !important; background-color: white !important; margin: 0 auto; }
        @media print {.no-print { display: none !important; } }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // 設定檔
        const ACTIVITY_TYPES = ['Food', 'Shopping', 'Sightseeing', 'Transport', 'Hotel'];
        const TYPE_LABEL_COLORS = { 'Food': 'bg-orange-200 text-orange-700', 'Shopping': 'bg-pink-200 text-pink-700', 'Sightseeing': 'bg-purple-200 text-purple-700', 'Transport': 'bg-blue-200 text-blue-700', 'Hotel': 'bg-green-200 text-green-700' };
        const TYPE_CARD_COLORS = { 'Food': 'bg-orange-50 border-orange-100', 'Shopping': 'bg-pink-50 border-pink-100', 'Sightseeing': 'bg-purple-50 border-purple-100', 'Transport': 'bg-blue-50 border-blue-100', 'Hotel': 'bg-green-50 border-green-100' };

        // 完整原始 5 天行程數據
        const INITIAL_DATA = [
            { id: '1', day: 1, time: '15:10', type: 'Transport', location: '關西機場 (KIX)', note: '下機、過關及取行李', cost: 0, addedBy: 'System' },
            { id: '2', day: 1, time: '16:35', type: 'Transport', location: '機場至市區', note: '南海電鐵 Rapi:t (約38分鐘)', cost: 1450, addedBy: 'System' },
            { id: '3', day: 1, time: '17:30', type: 'Hotel', location: 'Mimaru Osaka Namba North', note: '辦理入住手續', cost: 0, addedBy: 'System' },
            { id: '4', day: 1, time: '18:30', type: 'Sightseeing', location: '大阪天滿宮 (天神祭)', note: '宵宮屋台掃街，感受到地祭典氣氛', cost: 3000, addedBy: 'System' },
            { id: '5', day: 2, time: '10:00', type: 'Transport', location: '前往 USJ', note: '難波站搭乘阪神/JR線', cost: 370, addedBy: 'System' },
            { id: '6', day: 2, time: '11:00', type: 'Sightseeing', location: '日本環球影城', note: '重點：任天堂、大金剛、咒術迴戰', cost: 8900, addedBy: 'System' },
            { id: '7', day: 2, time: '19:15', type: 'Food', location: 'Universal CityWalk', note: '晚餐：Shake Shack 或 Red Lobster', cost: 4500, addedBy: 'System' },
            { id: '8', day: 3, time: '10:30', type: 'Shopping', location: '心齋橋筋商店街', note: 'Pet Paradise 寵物衫及飾物', cost: 15000, addedBy: 'System' },
            { id: '9', day: 3, time: '12:30', type: 'Food', location: '難波區午餐', note: '一蘭拉麵或本家章魚燒', cost: 1500, addedBy: 'System' },
            { id: '10', day: 3, time: '14:00', type: 'Shopping', location: '日本橋 (Den Den Town)', note: '模型、遊戲、動漫周邊巡禮', cost: 20000, addedBy: 'System' },
            { id: '11', day: 3, time: '18:30', type: 'Food', location: '蟹奉行 (千日前店)', note: '90分鐘松葉蟹放題 (任食)', cost: 7000, addedBy: 'System' },
            { id: '12', day: 4, time: '10:15', type: 'Sightseeing', location: '法善寺', note: '參拜水掛不動尊、遊覽橫丁', cost: 0, addedBy: 'System' },
            { id: '13', day: 4, time: '11:30', type: 'Food', location: '「喝鈍」豬扒飯', note: '法善寺旁地道美食', cost: 1200, addedBy: 'System' },
            { id: '14', day: 4, time: '13:00', type: 'Shopping', location: '大丸梅田店 13F', note: 'Nintendo, Pokémon, Capcom 專賣店', cost: 12000, addedBy: 'System' },
            { id: '15', day: 4, time: '17:30', type: 'Sightseeing', location: '梅田藍天大廈', note: '空中庭園展望台看日落 (強冷氣)', cost: 1500, addedBy: 'System' },
            { id: '16', day: 4, time: '19:00', type: 'Food', location: '馳走三昧 (大丸14F)', note: '蟹腳及海鮮自助餐', cost: 6500, addedBy: 'System' },
            { id: '17', day: 5, time: '11:00', type: 'Shopping', location: '難波 Parks 5F', note: 'P2 Dog & Cat 最後採購', cost: 8000, addedBy: 'System' },
            { id: '18', day: 5, time: '12:30', type: 'Food', location: '肉處 倉', note: '午餐：高質燒肉', cost: 4000, addedBy: 'System' },
            { id: '19', day: 5, time: '14:30', type: 'Transport', location: '南海電鐵 Rapi:t', note: '前往機場', cost: 1450, addedBy: 'System' },
            { id: '20', day: 5, time: '18:30', type: 'Transport', location: '關西機場 CX595', note: '起飛返港 (記得取 The Ginza)', cost: 0, addedBy: 'System' }
        ];

        function App() {
            const [userName, setUserName] = useState(localStorage.getItem('travel_user_name') || '');
            const [activities, setActivities] = useState(() => {
                const saved = localStorage.getItem('osaka_trip_data');
                return saved ? JSON.parse(saved) : INITIAL_DATA;
            });
            const [viewMode, setViewMode] = useState('day');
            const [activeDay, setActiveDay] = useState(1);
            const [activeCategory, setActiveCategory] = useState('Food');
            const [isModalOpen, setIsModalOpen] = useState(false);
            const [editingActivity, setEditingActivity] = useState(null);

            // 儲存數據到本地 (Standalone版本)
            useEffect(() => {
                localStorage.setItem('osaka_trip_data', JSON.stringify(activities));
            }, [activities]);

            const filteredList = activities
               .filter(a => viewMode === 'day' ? a.day === activeDay : a.type === activeCategory)
               .sort((a, b) => viewMode === 'day' ? a.time.localeCompare(b.time) : a.day - b.day);

            const handleExportPDF = () => {
                const element = document.getElementById('itinerary-content');
                element.classList.add('pdf-mode');
                const opt = { 
                    margin: 0, 
                    filename: `Osaka_Day${activeDay}.pdf`, 
                    image: { type: 'jpeg', quality: 0.98 }, 
                    html2canvas: { scale: 2 }, 
                    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' } 
                };
                html2pdf().set(opt).from(element).save().then(() => element.classList.remove('pdf-mode'));
            };

            const handleSave = (e) => {
                e.preventDefault();
                const fd = new FormData(e.target);
                const newData = {
                    id: editingActivity ? editingActivity.id : Date.now().toString(),
                    day: parseInt(fd.get('day')),
                    time: fd.get('time'),
                    type: fd.get('type'),
                    location: fd.get('location'),
                    note: fd.get('note'),
                    cost: parseInt(fd.get('cost')) || 0,
                    addedBy: editingActivity?.addedBy || userName
                };

                if (editingActivity) {
                    setActivities(activities.map(a => a.id === editingActivity.id ? newData : a));
                } else {
                    setActivities([...activities, newData]);
                }
                setIsModalOpen(false);
                setEditingActivity(null);
            };

            if (!userName) {
                return (
                    <div className="flex items-center justify-center h-screen p-6">
                        <div className="bg-white rounded-[2.5rem] p-10 w-full max-w-sm shadow-2xl text-center border-b-8 border-slate-900">
                            <h2 className="text-2xl font-black mb-8 text-slate-900 uppercase">Who is here?</h2>
                            <div className="space-y-3">
                                {['Donnie', 'Kim', '阮樂'].map(n => (
                                    <button key={n} onClick={() => {setUserName(n); localStorage.setItem('travel_user_name', n);}} className="w-full py-4 bg-slate-900 text-white hover:bg-blue-600 rounded-2xl font-bold transition-all shadow-md">{n}</button>
                                ))}
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="max-w-5xl mx-auto bg-white min-h-screen shadow-2xl flex flex-col md:flex-row overflow-hidden">
                    {/* 側邊導覽 */}
                    <aside className="no-print w-full md:w-32 bg-slate-900 flex flex-row md:flex-col items-center py-8 overflow-x-auto no-scrollbar">
                        <div className="flex flex-col gap-2 mb-6 w-full px-4">
                            <button onClick={() => setViewMode('day')} className={`py-3 text-[10px] font-black rounded-xl ${viewMode === 'day' ? 'bg-blue-600 text-white' : 'bg-slate-800 text-slate-500'}`}>BY DAY</button>
                            <button onClick={() => setViewMode('category')} className={`py-3 text-[10px] font-black rounded-xl ${viewMode === 'category' ? 'bg-blue-600 text-white' : 'bg-slate-800 text-slate-500'}`}>BY TYPE</button>
                        </div>
                        <div className="flex flex-row md:flex-col space-x-2 md:space-x-0 md:space-y-4 px-4">
                            {viewMode === 'day' ? [1, 2, 3, 4, 5].map(d => (
                                <button key={d} onClick={() => setActiveDay(d)} className={`w-12 h-12 md:w-20 md:h-20 rounded-3xl flex items-center justify-center transition-all ${activeDay === d ? 'bg-blue-600 text-white shadow-lg' : 'bg-slate-800 text-slate-400'}`}>D{d}</button>
                            )) : ACTIVITY_TYPES.map(t => (
                                <button key={t} onClick={() => setActiveCategory(t)} className={`w-12 h-12 md:w-20 md:h-20 rounded-3xl flex items-center justify-center text-[8px] font-black ${activeCategory === t ? 'bg-blue-600 text-white shadow-lg' : 'bg-slate-800 text-slate-400'}`}>{t==='Food'?'REST':t}</button>
                            ))}
                        </div>
                        <div className="mt-auto pt-8 no-print">
                             <button onClick={() => {localStorage.removeItem('travel_user_name'); window.location.reload();}} className="text-[10px] text-slate-500 hover:text-white font-bold">LOGOUT</button>
                        </div>
                    </aside>

                    {/* 主內容區 */}
                    <main id="itinerary-content" className="flex-1 p-6 md:p-10" style={{ backgroundColor: '#ADD8E6' }}>
                        <header className="flex justify-between items-start mb-8">
                            <div>
                                <h1 className="text-3xl font-black text-slate-900 uppercase">Osaka 2026</h1>
                                <p className="text-slate-900 font-bold text-xs uppercase mt-2">{viewMode === 'day' ? `Day ${activeDay}` : activeCategory}</p>
                            </div>
                            <div className="flex gap-2 no-print">
                                <button onClick={handleExportPDF} className="bg-white text-slate-900 border-2 border-slate-900 px-4 py-3 rounded-2xl font-black text-[10px] shadow-xl hover:bg-slate-50">PDF</button>
                                <button onClick={() => {setEditingActivity(null); setIsModalOpen(true)}} className="bg-slate-900 text-white px-5 py-3 rounded-2xl font-black text-[10px] shadow-xl hover:bg-slate-800">+ ADD</button>
                            </div>
                        </header>

                        <div className="space-y-4">
                            {filteredList.map(act => (
                                <div key={act.id} className={`group rounded-[2rem] p-7 border-2 shadow-sm transition-all ${TYPE_CARD_COLORS[act.type]}`}>
                                    <div className="flex justify-between items-start">
                                        <div className="flex gap-5 flex-1">
                                            <div className="text-center min-w-[75px]">
                                                <p className="text-2xl font-black text-slate-900">{viewMode==='day' ? act.time : `D${act.day}`}</p>
                                                <span className={`text-[9px] font-black px-2 py-0.5 rounded-md uppercase ${TYPE_LABEL_COLORS[act.type]}`}>{act.type}</span>
                                            </div>
                                            <div>
                                                <h4 className="font-black text-slate-900 text-xl leading-tight">{act.location}</h4>
                                                <p className="text-sm text-slate-700 mt-2 font-medium">{act.note}</p>
                                                <div className="flex items-center gap-2 mt-4">
                                                    <span className="text-sm font-black text-blue-800 uppercase tracking-tighter">BY {act.addedBy}</span>
                                                </div>
                                            </div>
                                        </div>
                                        <div className="flex gap-1 no-print">
                                            <button onClick={() => {setEditingActivity(act); setIsModalOpen(true)}} className="p-2 text-slate-400 hover:text-blue-600 transition-colors"><svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" /></svg></button>
                                            <button onClick={() => {if(confirm('Delete?')) setActivities(activities.filter(a => a.id !== act.id))}} className="p-2 text-slate-400 hover:text-red-600 transition-colors"><svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg></button>
                                        </div>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </main>

                    {/* 編輯視窗 */}
                    {isModalOpen && (
                        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-md no-print">
                            <form onSubmit={handleSave} className="bg-white rounded-[3rem] p-10 w-full max-w-lg shadow-2xl">
                                <h3 className="text-2xl font-black text-slate-900 mb-8 uppercase">Edit Details</h3>
                                <div className="space-y-4">
                                    <div className="grid grid-cols-2 gap-4">
                                        <div className="col-span-1">
                                            <label className="text-[10px] font-black text-slate-400 uppercase ml-2 block mb-1">Day</label>
                                            <select name="day" defaultValue={editingActivity?.day || activeDay} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black">
                                                {[1, 2, 3, 4, 5].map(d => <option key={d} value={d}>Day {d}</option>)}
                                            </select>
                                        </div>
                                        <div className="col-span-1">
                                            <label className="text-[10px] font-black text-slate-400 uppercase ml-2 block mb-1">Time</label>
                                            <input name="time" type="time" required defaultValue={editingActivity?.time || "09:00"} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                        </div>
                                    </div>
                                    <div>
                                        <label className="text-[10px] font-black text-slate-400 uppercase ml-2 block mb-1">Type</label>
                                        <select name="type" defaultValue={editingActivity?.type || (viewMode === 'category' ? activeCategory : 'Sightseeing')} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black">
                                            {ACTIVITY_TYPES.map(t => <option key={t} value={t}>{t}</option>)}
                                        </select>
                                    </div>
                                    <input name="location" placeholder="LOCATION" type="text" required defaultValue={editingActivity?.location || ""} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                    <textarea name="note" placeholder="NOTES" rows="3" defaultValue={editingActivity?.note || ""} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black"></textarea>
                                    <input name="cost" placeholder="BUDGET" type="number" defaultValue={editingActivity?.cost || 0} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                </div>
                                <div className="flex gap-4 mt-10">
                                    <button type="button" onClick={() => setIsModalOpen(false)} className="flex-1 py-4 text-gray-400 font-bold uppercase text-[10px]">Cancel</button>
                                    <button type="submit" className="flex-1 py-4 bg-slate-900 text-white rounded-[2rem] font-bold shadow-xl uppercase text-[10px]">Save</button>
                                </div>
                            </form>
                        </div>
                    )}
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
