<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ML Budget - 極簡記帳</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { background-color: #F2F2F7; font-family: -apple-system, BlinkMacSystemFont, sans-serif; -webkit-tap-highlight-color: transparent; }
        .ios-card { background: white; border-radius: 20px; box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
        .keypad-btn { background: white; border-radius: 15px; font-size: 24px; font-weight: 600; height: 60px; display: flex; align-items: center; justify-content: center; transition: all 0.1s; border: 1px solid #E5E5EA; }
        .keypad-btn:active { background: #E5E5EA; transform: scale(0.95); }
        .safe-bottom { padding-bottom: env(safe-area-inset-bottom); }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        function App() {
            const [view, setView] = useState('input'); // input, history, stats
            const [amount, setAmount] = useState('0');
            const [records, setRecords] = useState(() => JSON.parse(localStorage.getItem('ML_BUDGET_DATA') || '[]'));
            const [selectedCat, setSelectedCat] = useState('餐飲');
            
            const categories = [
                {n: '餐飲', i: '🍔', c: '#FF9500'},
                {n: '交通', i: '🚌', c: '#007AFF'},
                {n: '購物', i: '🛍️', c: '#AF52DE'},
                {n: '娛樂', i: '🎮', c: '#FF2D55'},
                {n: '居家', i: '🏠', c: '#34C759'},
                {n: '其他', i: '💰', c: '#8E8E93'}
            ];

            useEffect(() => {
                localStorage.setItem('ML_BUDGET_DATA', JSON.stringify(records));
            }, [records]);

            const handleNumber = (num) => {
                setAmount(prev => prev === '0' ? String(num) : prev + num);
            };

            const handleClear = () => setAmount('0');

            const handleSave = () => {
                if (amount === '0') return;
                const newRecord = {
                    id: Date.now(),
                    amount: parseFloat(amount),
                    category: selectedCat,
                    date: new Date().toLocaleDateString(),
                    timestamp: new Date().getTime()
                };
                setRecords([newRecord, ...records]);
                setAmount('0');
                alert('已記錄成功！');
            };

            return (
                <div className="max-w-md mx-auto min-h-screen flex flex-col relative">
                    {/* Header */}
                    <div className="pt-14 px-6 pb-4">
                        <h1 className="text-3xl font-black text-gray-900">ML Budget</h1>
                    </div>

                    {/* Main Content */}
                    <div className="flex-1 px-6">
                        {view === 'input' && (
                            <div className="space-y-6">
                                {/* Display */}
                                <div className="ios-card p-6 text-right">
                                    <div className="text-xs font-bold text-gray-400 uppercase mb-1">本次支出金額 (HKD)</div>
                                    <div className="text-5xl font-black text-blue-600 truncate">$ {amount}</div>
                                </div>

                                {/* Category Picker */}
                                <div className="grid grid-cols-3 gap-3">
                                    {categories.map(cat => (
                                        <button 
                                            key={cat.n} 
                                            onClick={() => setSelectedCat(cat.n)}
                                            className={`p-3 rounded-2xl flex flex-col items-center gap-1 transition-all ${selectedCat === cat.n ? 'bg-blue-600 text-white shadow-lg scale-105' : 'bg-white text-gray-500'}`}
                                        >
                                            <span className="text-2xl">{cat.i}</span>
                                            <span className="text-xs font-bold">{cat.n}</span>
                                        </button>
                                    ))}
                                </div>

                                {/* Keypad */}
                                <div className="grid grid-cols-3 gap-3 pt-4">
                                    {[1, 2, 3, 4, 5, 6, 7, 8, 9, '.', 0].map(n => (
                                        <button key={n} onClick={() => handleNumber(n)} className="keypad-btn shadow-sm">{n}</button>
                                    ))}
                                    <button onClick={handleClear} className="keypad-btn text-red-500 shadow-sm">C</button>
                                    <button onClick={handleSave} className="col-span-3 bg-gray-900 text-white h-16 rounded-2xl font-black text-xl shadow-xl mt-2 active:scale-95 transition-all">確認儲存支出</button>
                                </div>
                            </div>
                        )}

                        {view === 'history' && (
                            <div className="space-y-4">
                                <h2 className="font-bold text-gray-400 px-1">最近記錄</h2>
                                {records.length === 0 && <div className="text-center py-20 text-gray-400">目前沒有記帳資料</div>}
                                {records.map(r => (
                                    <div key={r.id} className="ios-card p-4 flex justify-between items-center">
                                        <div className="flex items-center gap-3">
                                            <div className="w-10 h-10 bg-gray-50 rounded-full flex items-center justify-center text-lg">
                                                {categories.find(c => c.n === r.category)?.i}
                                            </div>
                                            <div>
                                                <div className="font-bold text-gray-800">{r.category}</div>
                                                <div className="text-[10px] text-gray-400">{r.date}</div>
                                            </div>
                                        </div>
                                        <div className="font-black text-lg text-red-500">-$ {r.amount}</div>
                                    </div>
                                ))}
                            </div>
                        )}
                    </div>

                    {/* Tab Bar */}
                    <div className="bg-white/80 backdrop-blur-md border-t border-gray-100 px-10 py-4 flex justify-between safe-bottom sticky bottom-0">
                        <TabIcon active={view === 'input'} icon="➕" label="記帳" onClick={() => setView('input')} />
                        <TabIcon active={view === 'history'} icon="📜" label="明細" onClick={() => setView('history')} />
                        <TabIcon active={view === 'stats'} icon="📊" label="分析" onClick={() => alert('分析功能開發中！')} />
                    </div>
                </div>
            );
        }

        const TabIcon = ({ active, icon, label, onClick }) => (
            <button onClick={onClick} className={`flex flex-col items-center gap-1 ${active ? 'text-blue-600' : 'text-gray-300'}`}>
                <span className="text-xl">{icon}</span>
                <span className="text-[10px] font-bold">{label}</span>
            </button>
        );

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
