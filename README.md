# 2026osaka
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
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore-compat.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=SF+Pro+Display:wght@400;700;900&display=swap');
        body { font-family: 'SF Pro Display', -apple-system, sans-serif; background-color: #C2DFFF; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        @media print { .no-print { display: none !important; } }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        const firebaseConfig = JSON.parse(__firebase_config);
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'osaka-trip-final';

        const ACTIVITY_TYPES = ['Food', 'Sightseeing', 'Transport', 'Hotel'];
        const TYPE_LABEL_COLORS = { 'Food': 'bg-orange-200 text-orange-700', 'Sightseeing': 'bg-purple-200 text-purple-700', 'Transport': 'bg-blue-200 text-blue-700', 'Hotel': 'bg-green-200 text-green-700' };
        const TYPE_CARD_COLORS = { 'Food': 'bg-orange-50/90 border-orange-100', 'Sightseeing': 'bg-purple-50/90 border-purple-100', 'Transport': 'bg-blue-50/90 border-blue-100', 'Hotel': 'bg-green-50/90 border-green-100' };

        const INITIAL_PLAN = [
            { day: 1, time: '15:10', type: 'Transport', location: '關西機場 (KIX)', note: '下機、過關及取行李', cost: 0, addedBy: 'System' },
            { day: 1, time: '16:35', type: 'Transport', location: '機場至市區', note: '南海電鐵 Rapi:t (約38分鐘)', cost: 1450, addedBy: 'System' },
            { day: 1, time: '17:30', type: 'Hotel', location: 'Mimaru Osaka Namba North', note: '辦理入住手續', cost: 0, addedBy: 'System' },
            { day: 1, time: '18:30', type: 'Sightseeing', location: '大阪天滿宮 (天神祭)', note: '宵宮屋台掃街，感受到地祭典氣氛', cost: 3000, addedBy: 'System' },
            { day: 2, time: '10:00', type: 'Transport', location: '前往 USJ', note: '難波站搭乘阪神/JR線', cost: 370, addedBy: 'System' },
            { day: 2, time: '11:00', type: 'Sightseeing', location: '日本環球影城', note: '重點：任天堂、大金剛、咒術迴戰', cost: 8900, addedBy: 'System' },
            { day: 2, time: '19:15', type: 'Food', location: 'Universal CityWalk', note: '晚餐：Shake Shack 或 Red Lobster', cost: 4500, addedBy: 'System' },
            { day: 3, time: '10:30', type: 'Sightseeing', location: '心齋橋筋商店街', note: 'Pet Paradise 寵物衫及飾物', cost: 15000, addedBy: 'System' },
            { day: 3, time: '12:30', type: 'Food', location: '難波區午餐', note: '一蘭拉麵或本家章魚燒', cost: 1500, addedBy: 'System' },
            { day: 3, time: '14:00', type: 'Sightseeing', location: '日本橋 (Den Den Town)', note: '模型、遊戲、動漫周邊巡禮', cost: 20000, addedBy: 'System' },
            { day: 3, time: '18:30', type: 'Food', location: '蟹奉行 (千日前店)', note: '90分鐘松葉蟹放題 (任食)', cost: 7000, addedBy: 'System' },
            { day: 4, time: '10:15', type: 'Sightseeing', location: '法善寺', note: '參拜水掛不動尊、遊覽橫丁', cost: 0, addedBy: 'System' },
            { day: 4, time: '11:30', type: 'Food', location: '「喝鈍」豬扒飯', note: '法善寺旁地道美食', cost: 1200, addedBy: 'System' },
            { day: 4, time: '13:00', type: 'Sightseeing', location: '大丸梅田店 13F', note: 'Nintendo, Pokémon, Capcom 專賣店', cost: 12000, addedBy: 'System' },
            { day: 4, time: '17:30', type: 'Sightseeing', location: '梅田藍天大廈', note: '空中庭園展望台看日落 (強冷氣)', cost: 1500, addedBy: 'System' },
            { day: 4, time: '19:00', type: 'Food', location: '馳走三昧 (大丸14F)', note: '蟹腳及海鮮自助餐', cost: 6500, addedBy: 'System' },
            { day: 5, time: '11:00', type: 'Sightseeing', location: '難波 Parks 5F', note: 'P2 Dog & Cat 最後採購', cost: 8000, addedBy: 'System' },
            { day: 5, time: '12:30', type: 'Food', location: '肉處 倉', note: '午餐：高質燒肉', cost: 4000, addedBy: 'System' },
            { day: 5, time: '14:30', type: 'Transport', location: '南海電鐵 Rapi:t', note: '前往機場', cost: 1450, addedBy: 'System' },
            { day: 5, time: '18:30', type: 'Transport', location: '關西機場 CX595', note: '起飛返港 (記得取 The Ginza)', cost: 0, addedBy: 'System' }
        ];

        function App() {
            const [user, setUser] = useState(null);
            const [userName, setUserName] = useState(localStorage.getItem('travel_user_name') || '');
            const [activities, setActivities] = useState([]);
            const [activeDay, setActiveDay] = useState(1);
            const [isModalOpen, setIsModalOpen] = useState(false);
            const [editingActivity, setEditingActivity] = useState(null);

            useEffect(() => {
                const initAuth = async () => {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await auth.signInWithCustomToken(__initial_auth_token);
                    } else {
                        await auth.signInAnonymously();
                    }
                };
                initAuth();
                auth.onAuthStateChanged(setUser);
            }, []);

            useEffect(() => {
                if (!user) return;
                const colRef = db.collection('artifacts').doc(appId).collection('public').doc('data').collection('activities');
                const unsubscribe = colRef.onSnapshot(async (snapshot) => {
                    if (snapshot.empty) {
                        const batch = db.batch();
                        INITIAL_PLAN.forEach(item => batch.set(colRef.doc(), item));
                        await batch.commit();
                    } else {
                        const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                        setActivities(data);
                    }
                });
                return () => unsubscribe();
            }, [user]);

            const currentActivities = activities
                .filter(a => a.day === activeDay)
                .sort((a, b) => a.time.localeCompare(b.time));

            const getRouteLink = () => {
                if (currentActivities.length < 2) return null;
                const origin = currentActivities[0].location;
                const destination = currentActivities[currentActivities.length - 1].location;
                const waypoints = currentActivities.slice(1, -1).map(a => a.location).join('|');
                return `https://www.google.com/maps/dir/?api=1&origin=${encodeURIComponent(origin)}&destination=${encodeURIComponent(destination)}&waypoints=${encodeURIComponent(waypoints)}&travelmode=transit`;
            };

            const handleExportPDF = () => {
                const element = document.getElementById('itinerary-content');
                const opt = { margin: 10, filename: `Day${activeDay}_Itinerary.pdf`, image: { type: 'jpeg', quality: 0.98 }, html2canvas: { scale: 2 }, jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' } };
                html2pdf().set(opt).from(element).save();
            };

            const handleSave = async (e) => {
                e.preventDefault();
                const fd = new FormData(e.target);
                const data = { day: parseInt(fd.get('day')), time: fd.get('time'), type: fd.get('type'), location: fd.get('location'), note: fd.get('note'), cost: parseInt(fd.get('cost')) || 0, addedBy: editingActivity?.addedBy || userName };
                const col = db.collection('artifacts').doc(appId).collection('public').doc('data').collection('activities');
                if (editingActivity) await col.doc(editingActivity.id).update(data);
                else await col.add(data);
                setIsModalOpen(false);
                setEditingActivity(null);
            };

            if (!userName) {
                return (
                    <div className="flex items-center justify-center h-screen p-6">
                        <div className="bg-white rounded-[2.5rem] p-10 w-full max-w-sm shadow-2xl text-center border-b-8 border-slate-900">
                            <h2 className="text-2xl font-black mb-2 text-slate-900 leading-tight">規劃人員登錄</h2>
                            <div className="space-y-3 mt-8">
                                {['Donnie', 'Kim', '阮樂'].map(n => (
                                    <button key={n} onClick={() => {setUserName(n); localStorage.setItem('travel_user_name', n);}} className="w-full py-4 bg-slate-900 text-white hover:bg-blue-600 rounded-2xl font-bold transition-all shadow-md">{n}</button>
                                ))}
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="max-w-4xl mx-auto bg-white min-h-screen shadow-2xl flex flex-col md:flex-row overflow-hidden">
                    <aside className="no-print w-full md:w-28 bg-slate-900 flex flex-row md:flex-col items-center py-8 overflow-x-auto no-scrollbar">
                        <div className="flex flex-row md:flex-col space-x-3 md:space-x-0 md:space-y-6 px-4 md:px-0">
                            {[1, 2, 3, 4, 5].map(d => (
                                <button key={d} onClick={() => setActiveDay(d)} className={`w-14 h-14 md:w-20 md:h-20 rounded-3xl flex flex-col items-center justify-center transition-all ${activeDay === d ? 'bg-blue-600 text-white shadow-lg' : 'bg-slate-800 text-slate-400'}`}>
                                    <span className="text-[10px] font-black opacity-60">DAY</span>
                                    <span className="text-xl font-black">{d}</span>
                                </button>
                            ))}
                        </div>
                    </aside>

                    <main id="itinerary-content" className="flex-1 p-6 md:p-10" style={{ backgroundColor: '#ADD8E6' }}>
                        <header className="flex justify-between items-start mb-10">
                            <div>
                                <h1 className="text-3xl font-black text-slate-900">OSAKA 2026</h1>
                                <p className="text-slate-700 text-sm font-bold mt-2 no-print">身分: {userName} <button onClick={() => {localStorage.removeItem('travel_user_name'); location.reload()}} className="text-blue-700 underline ml-2">切換</button></p>
                            </div>
                            <div className="flex gap-2 no-print">
                                <button onClick={handleExportPDF} className="bg-white text-slate-900 border-2 border-slate-900 px-4 py-4 rounded-2xl font-black text-sm shadow-xl flex items-center gap-2">PDF</button>
                                <button onClick={() => {setEditingActivity(null); setIsModalOpen(true)}} className="bg-slate-900 text-white px-6 py-4 rounded-2xl font-black text-sm shadow-xl">+ ADD</button>
                            </div>
                        </header>

                        <div className="space-y-6">
                            {currentActivities.map(act => (
                                <div key={act.id} className={`group relative rounded-[2.5rem] p-7 border-2 shadow-sm transition-all ${TYPE_CARD_COLORS[act.type]}`}>
                                    <div className="flex justify-between items-start">
                                        <div className="flex gap-5">
                                            <div className="text-center min-w-[70px]">
                                                <p className="text-2xl font-black text-slate-900">{act.time}</p>
                                                <span className={`text-[10px] font-black px-2 py-0.5 rounded-lg uppercase ${TYPE_LABEL_COLORS[act.type]}`}>{act.type}</span>
                                            </div>
                                            <div>
                                                <h4 className="font-black text-slate-900 text-xl">{act.location}</h4>
                                                <p className="text-sm text-slate-700 mt-2 font-medium">{act.note}</p>
                                                <div className="flex items-center gap-3 mt-4">
                                                    <span className="text-sm font-black text-blue-800">¥{act.cost.toLocaleString()}</span>
                                                    <div className="flex items-center gap-1.5 bg-white/60 px-3 py-1.5 rounded-full border border-slate-100">
                                                        <div className={`w-4 h-4 rounded-full flex items-center justify-center text-[8px] text-white font-black ${act.addedBy === 'Donnie' ? 'bg-red-500' : act.addedBy === 'Kim' ? 'bg-blue-500' : 'bg-green-500'}`}>{act.addedBy ? act.addedBy.charAt(0) : '?'}</div>
                                                        <span className="text-[10px] text-slate-500 font-black uppercase">BY {act.addedBy}</span>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                        <div className="flex gap-1 no-print">
                                            <button onClick={() => {setEditingActivity(act); setIsModalOpen(true)}} className="p-2 text-slate-400 hover:text-blue-600"><svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" /></svg></button>
                                            <button onClick={async () => confirm('刪除？') && await db.collection('artifacts').doc(appId).collection('public').doc('data').collection('activities').doc(act.id).delete()} className="p-2 text-slate-400 hover:text-red-600"><svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg></button>
                                        </div>
                                    </div>
                                </div>
                            ))}

                            {/* Daily Route Section */}
                            {currentActivities.length > 1 && (
                                <div className="mt-12 bg-slate-900 rounded-[2.5rem] p-8 text-white shadow-2xl">
                                    <h3 className="text-xl font-black mb-4 flex items-center gap-2">
                                        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7" /></svg>
                                        Day {activeDay} 路線導航
                                    </h3>
                                    <div className="space-y-2 mb-6 opacity-80 text-sm">
                                        {currentActivities.map((act, i) => (
                                            <div key={act.id} className="flex items-center gap-2">
                                                <span className="w-5 h-5 bg-blue-600 rounded-full flex items-center justify-center text-[10px] font-black">{i+1}</span>
                                                <span>{act.location}</span>
                                            </div>
                                        ))}
                                    </div>
                                    <a href={getRouteLink()} target="_blank" className="block w-full bg-blue-600 hover:bg-blue-500 text-white text-center py-4 rounded-2xl font-black transition-all shadow-lg">
                                        在 Google Maps 開啟完整路徑
                                    </a>
                                </div>
                            )}
                        </div>
                    </main>

                    {isModalOpen && (
                        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-md">
                            <form onSubmit={handleSave} className="bg-white rounded-[3rem] p-10 w-full max-w-lg shadow-2xl">
                                <h3 className="text-2xl font-bold mb-8">行程建議</h3>
                                <div className="space-y-4">
                                    <div className="grid grid-cols-2 gap-4">
                                        <input name="day" type="hidden" value={editingActivity?.day || activeDay} />
                                        <div className="col-span-1">
                                            <label className="text-[10px] font-black text-slate-400 uppercase ml-2">時間</label>
                                            <input name="time" type="time" required defaultValue={editingActivity?.time || "09:00"} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                        </div>
                                        <div className="col-span-1">
                                            <label className="text-[10px] font-black text-slate-400 uppercase ml-2">類型</label>
                                            <select name="type" defaultValue={editingActivity?.type || 'Sightseeing'} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black">
                                                {ACTIVITY_TYPES.map(t => <option key={t} value={t}>{t}</option>)}
                                            </select>
                                        </div>
                                    </div>
                                    <input name="location" placeholder="地點" type="text" required defaultValue={editingActivity?.location || ""} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                    <textarea name="note" placeholder="備註" rows="3" defaultValue={editingActivity?.note || ""} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black"></textarea>
                                    <input name="cost" placeholder="預算 (JPY)" type="number" defaultValue={editingActivity?.cost || 0} className="w-full bg-slate-50 border-none rounded-2xl p-4 text-sm font-black" />
                                </div>
                                <div className="flex gap-4 mt-10">
                                    <button type="button" onClick={() => setIsModalOpen(false)} className="flex-1 py-4 text-gray-400 font-bold">取消</button>
                                    <button type="submit" className="flex-1 py-4 bg-slate-900 text-white rounded-[2rem] font-bold shadow-xl">發佈</button>
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
