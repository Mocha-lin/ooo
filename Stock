<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StockMaster 分析儀</title>
    <!-- 引入 Tailwind CSS 進行快速樣式設定 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 Chart.js 繪製圖表 -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- 引入 FontAwesome 圖示 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'beige-bg': '#F7F5F0', // 淺米白背景
                        'beige-sidebar': '#EBE8E1', // 深一點的米色側邊欄
                        'text-primary': '#4A453C', // 深灰棕色文字
                        'text-secondary': '#8C8578',
                        'accent-red': '#D64545',
                        'accent-green': '#3E9158',
                        'card-bg': '#FFFFFF'
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #F7F5F0;
            color: #4A453C;
            overflow: hidden; /* 防止整個頁面捲動 */
        }
        
        /* 滾動條美化 */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1; 
        }
        ::-webkit-scrollbar-thumb {
            background: #ccc; 
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #b3b3b3; 
        }

        /* 風險預警動畫 */
        @keyframes flash-red {
            0% { box-shadow: 0 0 0 0 rgba(214, 69, 69, 0.7); border-color: #D64545; }
            70% { box-shadow: 0 0 0 10px rgba(214, 69, 69, 0); border-color: #D64545; }
            100% { box-shadow: 0 0 0 0 rgba(214, 69, 69, 0); border-color: transparent; }
        }
        @keyframes flash-green {
            0% { box-shadow: 0 0 0 0 rgba(62, 145, 88, 0.7); border-color: #3E9158; }
            70% { box-shadow: 0 0 0 10px rgba(62, 145, 88, 0); border-color: #3E9158; }
            100% { box-shadow: 0 0 0 0 rgba(62, 145, 88, 0); border-color: transparent; }
        }

        .alert-overbought {
            animation: flash-red 2s infinite;
            border: 2px solid #D64545;
        }
        .alert-oversold {
            animation: flash-green 2s infinite;
            border: 2px solid #3E9158;
        }
        
        /* 拖曳時的樣式 */
        .dragging {
            opacity: 0.5;
            background-color: #e0ded6;
        }
    </style>
</head>
<body class="h-screen flex text-base">

    <!-- 左側欄 (1/6) -->
    <aside class="w-1/6 min-w-[250px] bg-beige-sidebar flex flex-col border-r border-gray-300 shadow-sm z-10 transition-all duration-300" id="sidebar">
        <!-- 標題與工具列 -->
        <div class="p-4 border-b border-gray-300 bg-beige-sidebar sticky top-0">
            <h1 class="text-xl font-bold mb-4 tracking-wide text-text-primary"><i class="fas fa-chart-line mr-2"></i>StockMaster</h1>
            
            <!-- 搜尋欄 -->
            <div class="relative">
                <input type="text" id="searchInput" placeholder="搜尋股號或股名..." 
                       class="w-full pl-8 pr-3 py-2 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400 bg-white text-sm">
                <i class="fas fa-search absolute left-3 top-3 text-gray-400 text-xs"></i>
            </div>
            
            <div class="flex justify-between mt-3 text-xs text-text-secondary">
                <span>已存: <span id="stockCount">0</span> 檔</span>
                <button onclick="createNewEntry()" class="hover:text-text-primary underline cursor-pointer"><i class="fas fa-plus"></i> 新增/匯入</button>
            </div>
        </div>

        <!-- 股票清單 (可滾動) -->
        <div id="stockList" class="flex-1 overflow-y-auto p-2 space-y-4">
            <!-- 內容由 JS 生成 -->
        </div>
    </aside>

    <!-- 右側欄 (5/6) -->
    <main class="flex-1 bg-beige-bg h-full overflow-hidden flex flex-col relative">
        
        <!-- 頂部資訊列 (如果未選擇股票則隱藏) -->
        <header id="mainHeader" class="bg-card-bg px-6 py-4 shadow-sm flex justify-between items-center border-b border-gray-200 hidden">
            <div class="flex items-center gap-4">
                <h2 id="headerTitle" class="text-2xl font-bold text-text-primary"></h2>
                <span id="headerCategory" class="px-2 py-1 bg-gray-200 text-xs rounded text-gray-600"></span>
            </div>
            <div class="flex items-center gap-6">
                <div class="text-right">
                    <div id="headerPrice" class="text-2xl font-bold"></div>
                    <div id="headerChange" class="text-sm font-medium"></div>
                </div>
                <div id="headerRiskBadge" class="px-3 py-1 rounded text-white text-sm font-bold hidden"></div>
            </div>
        </header>

        <!-- 內容區域 (可滾動) -->
        <div id="contentArea" class="flex-1 overflow-y-auto p-6 scroll-smooth">
            
            <!-- 預設歡迎/匯入介面 -->
            <div id="importSection" class="max-w-3xl mx-auto mt-10 bg-card-bg p-8 rounded-xl shadow-md">
                <h2 class="text-xl font-bold mb-4">匯入分析資料</h2>
                <p class="text-sm text-gray-500 mb-2">請貼上由 ooo 模型產生的 JSON 代碼：</p>
                <textarea id="jsonInput" rows="10" class="w-full p-4 border border-gray-300 rounded-lg font-mono text-xs bg-gray-50 focus:ring-2 focus:ring-gray-400 outline-none" placeholder='{ "id": "2330", "name": "台積電"... }'></textarea>
                <div class="mt-4 flex justify-end gap-3">
                    <button onclick="clearImport()" class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-lg">清除</button>
                    <button onclick="processJSON()" class="px-6 py-2 bg-text-primary text-white rounded-lg hover:bg-gray-700 shadow-md">匯入並儲存</button>
                </div>
                <div id="importError" class="mt-3 text-red-500 text-sm hidden"></div>
            </div>

            <!-- 分析儀表板 (預設隱藏) -->
            <div id="dashboard" class="hidden space-y-6">
                
                <!-- 1. 摘要與風險預警區塊 -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- 左：K線與基本面 -->
                    <div class="bg-card-bg p-5 rounded-xl shadow-sm border border-gray-100">
                        <h3 class="font-bold text-lg mb-3 text-text-primary border-b pb-2">技術與籌碼概況</h3>
                        <div class="space-y-3 text-sm">
                            <div class="flex justify-between">
                                <span class="text-gray-500">K線狀態</span>
                                <span id="kLineStatus" class="font-bold"></span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-500">布林通道</span>
                                <span id="bollingerStatus" class="font-bold"></span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-500">股價淨值比 (PB)</span>
                                <span id="pbRatio" class="font-bold"></span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-500">資料更新</span>
                                <span id="updateTime" class="text-xs text-gray-400"></span>
                            </div>
                        </div>
                    </div>

                    <!-- 中：配息與預估 -->
                    <div class="md:col-span-2 bg-card-bg p-5 rounded-xl shadow-sm border border-gray-100">
                        <h3 class="font-bold text-lg mb-3 text-text-primary border-b pb-2">ROI 報酬率分析 (配息再投入)</h3>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <h4 class="text-xs font-bold text-gray-400 uppercase mb-1">2020至今表現</h4>
                                <p id="roiPast" class="text-sm leading-relaxed text-text-primary"></p>
                            </div>
                            <div class="bg-beige-bg p-3 rounded-lg">
                                <h4 class="text-xs font-bold text-accent-green uppercase mb-1"><i class="fas fa-bullseye"></i> 未來推估</h4>
                                <p id="roiProjected" class="text-sm leading-relaxed font-medium text-text-primary"></p>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 2. 圖表區塊 (多維度) -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- PE 直條圖 -->
                    <div class="bg-card-bg p-4 rounded-xl shadow-sm">
                        <h4 class="text-center font-bold mb-2 text-sm">本益比 (PE) 歷年趨勢</h4>
                        <canvas id="chartPE"></canvas>
                    </div>
                    <!-- 月營收折線圖 -->
                    <div class="bg-card-bg p-4 rounded-xl shadow-sm">
                        <h4 class="text-center font-bold mb-2 text-sm">月營收趨勢 (近12月)</h4>
                        <canvas id="chartRevenue"></canvas>
                    </div>
                    <!-- EPS 成長曲線 -->
                    <div class="bg-card-bg p-4 rounded-xl shadow-sm">
                        <h4 class="text-center font-bold mb-2 text-sm">EPS 成長曲線</h4>
                        <canvas id="chartEPS"></canvas>
                    </div>
                </div>

                <!-- 3. 詳細表格與備忘錄 -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- 左：除權息列表 -->
                    <div class="md:col-span-2 bg-card-bg p-5 rounded-xl shadow-sm">
                        <h3 class="font-bold text-lg mb-3 text-text-primary">近年除權息資訊</h3>
                        <div class="overflow-x-auto">
                            <table class="w-full text-sm text-left">
                                <thead class="bg-beige-bg text-gray-600">
                                    <tr>
                                        <th class="px-4 py-2 rounded-l-lg">年度</th>
                                        <th class="px-4 py-2">現金股利</th>
                                        <th class="px-4 py-2">股票股利</th>
                                        <th class="px-4 py-2 rounded-r-lg">除息交易日</th>
                                    </tr>
                                </thead>
                                <tbody id="dividendsTableBody">
                                    <!-- JS 插入 -->
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <!-- 右：備忘錄 -->
                    <div class="bg-card-bg p-5 rounded-xl shadow-sm flex flex-col">
                        <h3 class="font-bold text-lg mb-3 text-text-primary flex justify-between items-center">
                            備忘筆記
                            <span id="saveStatus" class="text-xs font-normal text-green-600 opacity-0 transition-opacity">已儲存</span>
                        </h3>
                        <textarea id="memoArea" class="flex-1 w-full p-3 bg-beige-bg rounded-lg border-none focus:ring-2 focus:ring-gray-300 resize-none text-sm" placeholder="在此輸入您的觀察筆記..."></textarea>
                    </div>
                </div>

                <div class="h-10"></div> <!-- 底部緩衝 -->
            </div>
        </div>
    </main>

    <script>
        // --- 變數與初始化 ---
        let stocks = [];
        let currentStockId = null;
        let charts = {}; // 儲存 Chart.js 實例

        // 初始化
        document.addEventListener('DOMContentLoaded', () => {
            loadFromStorage();
            renderSidebar();
            
            // 監聽備忘錄輸入以自動儲存
            document.getElementById('memoArea').addEventListener('input', (e) => {
                if(currentStockId) {
                    const stock = stocks.find(s => s.id === currentStockId);
                    if(stock) {
                        stock.user_memo = e.target.value;
                        showSaveStatus();
                        saveToStorage();
                    }
                }
            });

            // 監聽搜尋
            document.getElementById('searchInput').addEventListener('input', (e) => {
                renderSidebar(e.target.value);
            });
        });

        // --- 資料管理 ---
        function loadFromStorage() {
            const data = localStorage.getItem('stockMasterData');
            if (data) {
                stocks = JSON.parse(data);
            }
            document.getElementById('stockCount').innerText = stocks.length;
        }

        function saveToStorage() {
            localStorage.setItem('stockMasterData', JSON.stringify(stocks));
            document.getElementById('stockCount').innerText = stocks.length;
        }

        function showSaveStatus() {
            const el = document.getElementById('saveStatus');
            el.style.opacity = '1';
            setTimeout(() => { el.style.opacity = '0'; }, 1000);
        }

        // --- 核心邏輯：匯入 ---
        function createNewEntry() {
            currentStockId = null;
            document.getElementById('dashboard').classList.add('hidden');
            document.getElementById('mainHeader').classList.add('hidden');
            document.getElementById('importSection').classList.remove('hidden');
            document.getElementById('jsonInput').value = '';
            document.getElementById('importError').classList.add('hidden');
        }

        function processJSON() {
            const input = document.getElementById('jsonInput').value;
            const errorEl = document.getElementById('importError');
            
            try {
                // 嘗試解析 JSON
                // 有些 AI 可能會給 Markdown 格式，嘗試移除 ```json 和 ```
                let cleanInput = input.replace(/```json/g, '').replace(/```/g, '').trim();
                const data = JSON.parse(cleanInput);

                // 基本驗證
                if (!data.id || !data.name) throw new Error("JSON 缺少 id 或 name 欄位");

                // 處理舊資料 (更新或新增)
                const existingIndex = stocks.findIndex(s => s.id === data.id);
                
                // 保留原本的備忘錄 (如果是更新)
                let existingMemo = "";
                if(existingIndex !== -1) {
                    existingMemo = stocks[existingIndex].user_memo || "";
                    stocks[existingIndex] = data; // 更新
                    stocks[existingIndex].user_memo = existingMemo;
                } else {
                    data.user_memo = "";
                    stocks.push(data);
                }

                saveToStorage();
                renderSidebar();
                loadStockDetail(data.id); // 直接跳轉到該股票
                
            } catch (e) {
                errorEl.innerText = "格式錯誤: " + e.message;
                errorEl.classList.remove('hidden');
            }
        }

        function clearImport() {
            document.getElementById('jsonInput').value = '';
        }

        // --- 側邊欄渲染 (含分類與搜尋) ---
        function renderSidebar(filterText = '') {
            const listEl = document.getElementById('stockList');
            listEl.innerHTML = '';

            // 1. 分組
            const groups = {};
            const uncategorized = [];

            stocks.forEach(stock => {
                const match = !filterText || 
                              stock.id.includes(filterText) || 
                              stock.name.includes(filterText);
                if (!match) return;

                if (stock.category) {
                    if (!groups[stock.category]) groups[stock.category] = [];
                    groups[stock.category].push(stock);
                } else {
                    uncategorized.push(stock);
                }
            });

            // 2. 渲染函數
            const createItem = (stock) => {
                const div = document.createElement('div');
                div.className = `p-3 rounded-lg cursor-pointer hover:bg-white transition-colors border border-transparent hover:border-gray-200 group ${stock.id === currentStockId ? 'bg-white border-gray-300 shadow-sm' : ''}`;
                div.draggable = true; // 啟用拖曳
                div.onclick = () => loadStockDetail(stock.id);
                
                // 簡易漲跌提醒與風險標示
                let statusDot = '';
                let riskText = '';
                
                if (stock.risk_status === 'overbought') statusDot = '<span class="w-2 h-2 rounded-full bg-red-500 inline-block animate-pulse"></span>';
                else if (stock.risk_status === 'oversold') statusDot = '<span class="w-2 h-2 rounded-full bg-green-500 inline-block animate-pulse"></span>';
                
                // 價格顏色
                const priceClass = stock.change_text && stock.change_text.includes('+') ? 'text-red-600' : (stock.change_text && stock.change_text.includes('-') ? 'text-green-600' : 'text-gray-600');

                div.innerHTML = `
                    <div class="flex justify-between items-center mb-1">
                        <span class="font-bold text-text-primary">${stock.name} <span class="text-xs font-normal text-gray-500">${stock.id}</span></span>
                        ${statusDot}
                    </div>
                    <div class="flex justify-between items-center text-xs">
                        <span class="${priceClass} font-mono">${stock.current_price}</span>
                        <span class="text-gray-400 truncate max-w-[80px]">${stock.k_line_summary || ''}</span>
                    </div>
                `;
                
                // 拖曳事件
                div.addEventListener('dragstart', () => div.classList.add('dragging'));
                div.addEventListener('dragend', () => {
                    div.classList.remove('dragging');
                    saveOrderFromDom(); // 重新排序後儲存
                });
                
                return div;
            };

            // 3. 輸出分類
            for (const [category, groupStocks] of Object.entries(groups)) {
                const catDiv = document.createElement('div');
                catDiv.innerHTML = `<h3 class="text-xs font-bold text-gray-400 uppercase mt-2 mb-1 px-1">${category}</h3>`;
                listEl.appendChild(catDiv);
                const groupContainer = document.createElement('div');
                groupContainer.className = "space-y-1";
                // 允許分類內的拖曳 (簡單實作)
                groupContainer.addEventListener('dragover', e => {
                    e.preventDefault();
                    const dragging = document.querySelector('.dragging');
                    groupContainer.appendChild(dragging);
                });
                groupStocks.forEach(s => groupContainer.appendChild(createItem(s)));
                listEl.appendChild(groupContainer);
            }

            // 4. 輸出未分類
            if (uncategorized.length > 0) {
                if (Object.keys(groups).length > 0) {
                    listEl.innerHTML += `<h3 class="text-xs font-bold text-gray-400 uppercase mt-4 mb-1 px-1">未分類</h3>`;
                }
                const uncategorizedContainer = document.createElement('div');
                uncategorizedContainer.className = "space-y-1";
                uncategorizedContainer.addEventListener('dragover', e => {
                    e.preventDefault();
                    const dragging = document.querySelector('.dragging');
                    uncategorizedContainer.appendChild(dragging);
                });
                uncategorized.forEach(s => uncategorizedContainer.appendChild(createItem(s)));
                listEl.appendChild(uncategorizedContainer);
            }
        }

        // 簡單的重新排序邏輯 (基於 DOM 順序重建 stocks 陣列，這裡簡化處理，僅針對未分類或單一列表有效，完整分類拖曳需更複雜邏輯)
        function saveOrderFromDom() {
            // 這個簡單版本僅觸發重新渲染，實際排序需配合 ID 查找
            // 由於分類結構複雜，此處暫不改變原始資料結構的順序，僅做 UI 互動展示
        }

        // --- 詳細頁面渲染 ---
        function loadStockDetail(id) {
            const stock = stocks.find(s => s.id === id);
            if (!stock) return;

            currentStockId = id;
            
            // UI 切換
            document.getElementById('importSection').classList.add('hidden');
            document.getElementById('dashboard').classList.remove('hidden');
            document.getElementById('mainHeader').classList.remove('hidden');

            // 1. Header 資訊
            document.getElementById('headerTitle').innerText = `${stock.name} (${stock.id})`;
            document.getElementById('headerCategory').innerText = stock.category || '未分類';
            document.getElementById('headerPrice').innerText = stock.current_price;
            
            const changeEl = document.getElementById('headerChange');
            changeEl.innerText = stock.change_text || '-';
            changeEl.className = "text-sm font-medium " + (stock.change_text?.includes('+') ? 'text-accent-red' : (stock.change_text?.includes('-') ? 'text-accent-green' : 'text-gray-500'));

            // 風險標示
            const riskBadge = document.getElementById('headerRiskBadge');
            const dashboardArea = document.querySelector('.bg-card-bg'); // 第一個卡片用於閃爍

            // 移除舊的閃爍
            const techCard = document.querySelector('#dashboard > div > div:first-child');
            techCard.classList.remove('alert-overbought', 'alert-oversold');
            
            riskBadge.classList.add('hidden');
            if (stock.risk_status === 'overbought') {
                riskBadge.innerText = '⚠️ 布林超買';
                riskBadge.className = 'px-3 py-1 rounded text-white text-sm font-bold bg-accent-red block';
                techCard.classList.add('alert-overbought');
            } else if (stock.risk_status === 'oversold') {
                riskBadge.innerText = '⚡ 布林破底';
                riskBadge.className = 'px-3 py-1 rounded text-white text-sm font-bold bg-accent-green block';
                techCard.classList.add('alert-oversold');
            }

            // 2. 摘要區塊
            document.getElementById('kLineStatus').innerText = stock.k_line_summary;
            document.getElementById('bollingerStatus').innerText = stock.risk_status === 'normal' ? '正常區間' : (stock.risk_status === 'overbought' ? '超買 (上軌)' : '破底 (下軌)');
            document.getElementById('pbRatio').innerText = stock.pb_ratio;
            document.getElementById('updateTime').innerText = stock.update_time;

            document.getElementById('roiPast').innerText = stock.analysis_content.roi_past;
            document.getElementById('roiProjected').innerText = stock.analysis_content.roi_projected;

            // 3. 表格
            const tbody = document.getElementById('dividendsTableBody');
            tbody.innerHTML = '';
            if (stock.analysis_content.dividends_table) {
                stock.analysis_content.dividends_table.forEach(row => {
                    const tr = document.createElement('tr');
                    tr.className = "border-b hover:bg-gray-50";
                    tr.innerHTML = `
                        <td class="px-4 py-3">${row.year}</td>
                        <td class="px-4 py-3">${row.cash}</td>
                        <td class="px-4 py-3">${row.stock}</td>
                        <td class="px-4 py-3 text-gray-500 text-xs">${row.date}</td>
                    `;
                    tbody.appendChild(tr);
                });
            }

            // 4. 備忘錄
            document.getElementById('memoArea').value = stock.user_memo || '';

            // 5. 圖表渲染
            renderCharts(stock.charts);

            // 更新側邊欄選中狀態
            renderSidebar(document.getElementById('searchInput').value);
        }

        // --- 圖表繪製 ---
        function renderCharts(chartData) {
            // 銷毀舊圖表
            if(charts['pe']) charts['pe'].destroy();
            if(charts['rev']) charts['rev'].destroy();
            if(charts['eps']) charts['eps'].destroy();

            // 通用設定
            Chart.defaults.font.family = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
            Chart.defaults.color = '#8C8578';

            // 1. PE Chart (Bar)
            const ctxPE = document.getElementById('chartPE').getContext('2d');
            charts['pe'] = new Chart(ctxPE, {
                type: 'bar',
                data: {
                    labels: chartData.pe_ratio.map(d => d.year),
                    datasets: [{
                        label: '本益比',
                        data: chartData.pe_ratio.map(d => d.value),
                        backgroundColor: '#EBE8E1',
                        hoverBackgroundColor: '#4A453C',
                        borderRadius: 4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: { legend: { display: false } },
                    scales: { y: { beginAtZero: false, grid: { display: false } }, x: { grid: { display: false } } }
                }
            });

            // 2. Revenue Chart (Line)
            const ctxRev = document.getElementById('chartRevenue').getContext('2d');
            charts['rev'] = new Chart(ctxRev, {
                type: 'line',
                data: {
                    labels: chartData.monthly_revenue.map(d => d.label),
                    datasets: [{
                        label: '營收(億)',
                        data: chartData.monthly_revenue.map(d => d.value),
                        borderColor: '#D64545',
                        backgroundColor: 'rgba(214, 69, 69, 0.1)',
                        fill: true,
                        tension: 0.3,
                        pointRadius: 2
                    }]
                },
                options: {
                    responsive: true,
                    plugins: { legend: { display: false } },
                    scales: { y: { grid: { color: '#f0f0f0' } }, x: { grid: { display: false } } }
                }
            });

            // 3. EPS Chart (Line/Bar mixed ideally, simple line here)
            const ctxEPS = document.getElementById('chartEPS').getContext('2d');
            charts['eps'] = new Chart(ctxEPS, {
                type: 'line',
                data: {
                    labels: chartData.eps_growth.map(d => d.period),
                    datasets: [{
                        label: 'EPS',
                        data: chartData.eps_growth.map(d => d.value),
                        borderColor: '#3E9158',
                        borderWidth: 2,
                        tension: 0.1,
                        pointBackgroundColor: '#fff',
                        pointBorderColor: '#3E9158',
                        pointRadius: 4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: { legend: { display: false } },
                    scales: { y: { grid: { borderDash: [5, 5] } } }
                }
            });
        }
    </script>
</body>
</html>
