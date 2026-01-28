<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>智能点单系统 - 备注增强版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700;900&display=swap');
        
        body { 
            font-family: 'Noto Sans SC', sans-serif; 
            background-color: #f8fafc; 
            -webkit-tap-highlight-color: transparent;
        }

        .scroll-hide::-webkit-scrollbar { display: none; }
        
        .tab-active {
            background-color: #2563eb !important;
            color: white !important;
            box-shadow: 0 4px 12px -2px rgba(37, 99, 235, 0.3);
        }

        .cart-item-active {
            border-color: #2563eb !important;
            background-color: #eff6ff !important;
            ring: 2px solid #2563eb;
        }

        .fade-in { animation: fadeIn 0.2s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }

        /* 独立看板页面容器 */
        #page-display {
            position: fixed;
            inset: 0;
            background: #ffffff;
            z-index: 1000;
            display: none;
            flex-direction: column;
        }
        
        .number-card {
            background: white;
            border-radius: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 900;
            border: 2px solid #f1f5f9;
        }

        .ready-animation {
            animation: pulse-green 2s infinite;
        }

        @keyframes pulse-green {
            0% { transform: scale(1); box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.4); }
            70% { transform: scale(1.05); box-shadow: 0 0 0 20px rgba(34, 197, 94, 0); }
            100% { transform: scale(1); box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); }
        }
    </style>
</head>
<body class="antialiased text-slate-900">

    <!-- 独立叫号看板页面 -->
    <div id="page-display">
        <div class="h-20 bg-white border-b flex items-center justify-between px-10 shadow-sm">
            <h1 class="text-2xl font-black text-slate-800">自助取餐看板</h1>
            <div class="flex items-center gap-4">
                <div id="display-time" class="text-xl font-bold text-slate-500">00:00:00</div>
                <button onclick="closeDisplayPage()" class="p-2 bg-slate-100 rounded-full font-bold">关闭</button>
            </div>
        </div>
        <div class="flex-grow p-8 grid grid-cols-12 gap-8 bg-slate-50 overflow-hidden">
            <div class="col-span-4 flex flex-col gap-4">
                <div class="bg-blue-600 rounded-2xl p-4 text-white font-bold">制作中 Preparing</div>
                <div id="screen-preparing" class="flex-grow bg-white rounded-3xl p-4 shadow-inner grid grid-cols-2 gap-3 content-start"></div>
            </div>
            <div class="col-span-8 flex flex-col gap-4">
                <div class="bg-green-500 rounded-2xl p-4 text-white font-bold text-center text-xl">请取餐 Please Pickup</div>
                <div id="screen-ready" class="flex-grow bg-white rounded-3xl p-6 shadow-inner grid grid-cols-3 gap-6 content-start"></div>
            </div>
        </div>
    </div>

    <div id="app" class="min-h-screen flex flex-col">
        <!-- 导航 -->
        <header class="bg-white p-4 sticky top-0 z-50 border-b border-slate-100 shadow-sm">
            <div class="max-w-7xl mx-auto flex flex-col gap-4">
                <div class="flex items-center justify-between">
                    <div class="flex items-center gap-2">
                        <div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center text-white font-black italic">T</div>
                        <h1 class="text-xl font-black text-slate-800">智慧点单系统</h1>
                    </div>
                    <button onclick="openDisplayPage()" class="text-xs bg-slate-900 text-white px-4 py-2 rounded-full font-bold">打开看板</button>
                </div>
                <div class="flex gap-2 overflow-x-auto scroll-hide">
                    <button onclick="switchTab('order')" id="btn-order" class="tab-btn px-5 py-2 rounded-xl text-sm font-bold bg-white text-slate-600 border border-slate-200 tab-active">点单收银</button>
                    <button onclick="switchTab('status')" id="btn-status" class="tab-btn px-5 py-2 rounded-xl text-sm font-bold bg-white text-slate-600 border border-slate-200">订单状态</button>
                    <button onclick="switchTab('manage')" id="btn-manage" class="tab-btn px-5 py-2 rounded-xl text-sm font-bold bg-white text-slate-600 border border-slate-200">系统设置</button>
                    <button onclick="switchTab('history')" id="btn-history" class="tab-btn px-5 py-2 rounded-xl text-sm font-bold bg-white text-slate-600 border border-slate-200">订单历史</button>
                </div>
            </div>
        </header>

        <main class="flex-grow p-4 max-w-7xl mx-auto w-full">
            <!-- 1. 点单收银 -->
            <div id="tab-order" class="tab-pane fade-in grid grid-cols-1 lg:grid-cols-12 gap-6">
                <!-- 商品区 -->
                <div class="lg:col-span-8 space-y-6">
                    <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100 min-h-[400px]">
                        <h2 class="text-[10px] font-black text-slate-400 mb-4 tracking-widest uppercase text-center">Select Products 商品列表</h2>
                        <div id="product-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4"></div>
                    </div>
                </div>

                <!-- 购物车区 -->
                <div class="lg:col-span-4 flex flex-col gap-4">
                    <div class="bg-white rounded-3xl shadow-sm border border-slate-100 flex flex-col h-[650px]">
                        <div class="p-5 border-b flex justify-between items-center bg-slate-50/50 rounded-t-3xl">
                            <h2 class="font-black text-slate-800">当前待下单</h2>
                            <button onclick="clearCart()" class="text-xs text-red-400 font-bold">清空</button>
                        </div>
                        
                        <!-- 待下单物品列表 -->
                        <div id="cart-list" class="flex-grow p-4 space-y-3 overflow-y-auto scroll-hide"></div>

                        <!-- 备注编辑区域 (整合手动输入与标签) -->
                        <div class="p-4 bg-slate-50 border-t border-slate-100">
                            <div class="flex items-center justify-between mb-2">
                                <h3 class="text-[10px] font-black text-slate-400 uppercase">Remark 备注编辑</h3>
                                <button onclick="applyToAll()" class="text-[9px] bg-slate-200 text-slate-600 px-2 py-0.5 rounded font-bold hover:bg-slate-300">应用至全单</button>
                            </div>
                            <!-- 手动输入框 -->
                            <input type="text" id="manual-note" 
                                   oninput="updateNoteFromInput()"
                                   placeholder="点击商品后输入备注或选择标签..." 
                                   class="w-full p-3 bg-white border border-slate-200 rounded-xl text-sm outline-none focus:ring-2 focus:ring-blue-500 mb-3 transition-all">
                            <!-- 快捷标签区 -->
                            <div id="tags-container" class="flex flex-wrap gap-2"></div>
                        </div>

                        <div class="p-5 space-y-4 border-t border-slate-100">
                            <div class="flex justify-between items-center font-black">
                                <span class="text-slate-400 text-xs">TOTAL 合计</span>
                                <span class="text-2xl text-blue-600 italic">¥<span id="cart-total">0.00</span></span>
                            </div>
                            <button onclick="submitOrder()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black text-lg shadow-lg hover:bg-blue-700 transition-all">
                                确认下单
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 2. 订单状态 -->
            <div id="tab-status" class="tab-pane hidden fade-in">
                <section class="bg-white rounded-3xl p-6 shadow-sm border border-blue-100">
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="font-black text-slate-800 text-xl">制作进度管理</h2>
                        <div class="text-[10px] text-slate-400 font-bold uppercase">实时同步至看板</div>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left">
                            <thead class="bg-slate-50 text-slate-400 text-xs font-bold uppercase">
                                <tr>
                                    <th class="p-4">订单号</th>
                                    <th class="p-4">商品详情</th>
                                    <th class="p-4 text-right">状态操作</th>
                                </tr>
                            </thead>
                            <tbody id="active-orders-body" class="divide-y divide-slate-50"></tbody>
                        </table>
                    </div>
                </section>
            </div>

            <!-- 3. 系统设置 -->
            <div id="tab-manage" class="tab-pane hidden fade-in space-y-6">
                <section class="bg-white rounded-3xl p-6 shadow-sm border border-slate-100">
                    <h2 class="font-black text-slate-800 mb-4">商品管理</h2>
                    <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-6">
                        <input type="text" id="m-name" placeholder="品名" class="p-3 bg-slate-50 rounded-xl outline-none border">
                        <input type="number" id="m-price" placeholder="单价" class="p-3 bg-slate-50 rounded-xl outline-none border">
                        <button onclick="addProduct()" class="bg-slate-900 text-white font-bold rounded-xl">添加</button>
                    </div>
                    <div id="product-manage-list" class="space-y-2"></div>
                </section>
                <section class="bg-white rounded-3xl p-6 shadow-sm border border-slate-100">
                    <h2 class="font-black text-slate-800 mb-4">标签管理</h2>
                    <div class="flex gap-3 mb-4">
                        <input type="text" id="tag-input" placeholder="输入新标签" class="flex-grow p-3 bg-slate-50 rounded-xl outline-none border">
                        <button onclick="addTag()" class="px-6 bg-slate-900 text-white font-bold rounded-xl">添加</button>
                    </div>
                    <div id="manage-tags-list" class="flex flex-wrap gap-2"></div>
                </section>
            </div>

            <!-- 4. 订单历史 -->
            <div id="tab-history" class="tab-pane hidden fade-in space-y-4">
                <div class="bg-white p-5 rounded-3xl flex justify-between items-center shadow-sm">
                    <h2 class="font-black text-slate-800">历史订单查询</h2>
                    <input type="date" id="history-date" onchange="loadHistory()" class="bg-slate-50 p-2 rounded-lg font-bold border-none text-sm">
                </div>
                <div class="bg-white rounded-3xl shadow-sm overflow-hidden">
                    <table class="w-full text-left">
                        <thead class="bg-slate-50 text-slate-400 text-[10px] uppercase font-black">
                            <tr>
                                <th class="p-4">单号</th>
                                <th class="p-4">内容</th>
                                <th class="p-4 text-right">总额</th>
                            </tr>
                        </thead>
                        <tbody id="history-body" class="divide-y divide-slate-50"></tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>

    <div id="toast-container" class="fixed top-10 left-1/2 -translate-x-1/2 z-[2000] w-max"></div>

    <script>
        // --- 数据存储 ---
        const DB = {
            get: (key, def = []) => JSON.parse(localStorage.getItem('tea_v6_' + key)) || def,
            set: (key, val) => localStorage.setItem('tea_v6_' + key, JSON.stringify(val)),
            nextNum: () => {
                let n = parseInt(localStorage.getItem('tea_v6_lastnum') || '0') + 1;
                if(n > 999) n = 1;
                localStorage.setItem('tea_v5_lastnum', n);
                return n;
            }
        };

        const STATUS_FLOW = ["制作中", "请取餐", "已完成"];
        const COLOR_MAP = { "制作中": "bg-blue-100 text-blue-600", "请取餐": "bg-green-100 text-green-600", "已完成": "bg-slate-800 text-white" };

        let products = DB.get('products', [{id:1, name:'招牌奶茶', price:18}, {id:2, name:'生椰拿铁', price:22}, {id:3, name:'手打柠檬茶', price:16}]);
        let tags = DB.get('tags', ["去冰", "少糖", "加珍珠", "多糖", "常温", "不要茶冻"]);
        let orders = DB.get('orders');
        let cart = []; 
        let selectedCartIndex = -1;

        // --- 核心切换 ---
        function switchTab(tab) {
            document.querySelectorAll('.tab-pane').forEach(p => p.classList.add('hidden'));
            document.getElementById('tab-' + tab).classList.remove('hidden');
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('tab-active'));
            document.getElementById('btn-' + tab).classList.add('tab-active');

            if(tab === 'order') { renderProducts(); renderTags(); renderCart(); }
            if(tab === 'status') { renderActiveOrders(); }
            if(tab === 'manage') { renderManageUI(); }
            if(tab === 'history') { loadHistory(); }
        }

        // --- 看板逻辑 ---
        function openDisplayPage() {
            document.getElementById('page-display').style.display = 'flex';
            renderScreenContent();
            window.displayInterval = setInterval(renderScreenContent, 3000);
            window.timeInterval = setInterval(() => {
                document.getElementById('display-time').innerText = new Date().toLocaleTimeString('zh-CN', {hour12:false});
            }, 1000);
        }
        function closeDisplayPage() {
            document.getElementById('page-display').style.display = 'none';
            clearInterval(window.displayInterval);
            clearInterval(window.timeInterval);
        }
        function renderScreenContent() {
            const prep = document.getElementById('screen-preparing');
            const ready = document.getElementById('screen-ready');
            prep.innerHTML = orders.filter(o => o.status === "制作中").slice(0, 12).map(o => `<div class="number-card h-16 text-2xl text-slate-400 animate-pulse">${o.num}</div>`).join('');
            ready.innerHTML = orders.filter(o => o.status === "请取餐").slice(0, 15).map(o => `<div class="number-card h-28 text-5xl text-green-600 border-green-200 ready-animation">${o.num}</div>`).join('');
        }

        // --- 点单逻辑 ---
        function renderProducts() {
            const grid = document.getElementById('product-grid');
            grid.innerHTML = products.map(p => `
                <div onclick="addToCart(${p.id})" class="bg-white p-4 rounded-2xl border-2 border-slate-100 hover:border-blue-400 active:scale-95 transition-all cursor-pointer text-center">
                    <div class="font-black text-slate-700 text-sm mb-1">${p.name}</div>
                    <div class="text-blue-500 font-black text-xs italic">¥${p.price.toFixed(2)}</div>
                </div>
            `).join('');
        }

        function addToCart(pid) {
            const p = products.find(x => x.id === pid);
            const existing = cart.find(i => i.id === pid && i.note === ""); 
            if (existing) {
                existing.count++;
                selectedCartIndex = cart.indexOf(existing);
            } else {
                cart.push({ ...p, note: "", count: 1 });
                selectedCartIndex = cart.length - 1;
            }
            renderCart();
            syncNoteInput();
        }

        function renderCart() {
            const list = document.getElementById('cart-list');
            let total = 0;
            list.innerHTML = cart.map((item, idx) => {
                total += item.price * item.count;
                const isActive = selectedCartIndex === idx;
                return `
                    <div onclick="selectCartItem(${idx})" class="p-4 rounded-2xl border transition-all cursor-pointer flex flex-col gap-1 ${isActive ? 'cart-item-active' : 'bg-slate-50 border-slate-100'}">
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-black text-slate-800">${item.name}</span>
                            <div class="flex items-center gap-2 bg-white px-2 py-1 rounded-lg border">
                                <button onclick="event.stopPropagation(); changeCount(${idx}, -1)" class="w-5 h-5 flex items-center justify-center font-bold text-slate-400">-</button>
                                <span class="text-xs font-black min-w-[1rem] text-center">${item.count}</span>
                                <button onclick="event.stopPropagation(); changeCount(${idx}, 1)" class="w-5 h-5 flex items-center justify-center font-bold text-blue-500">+</button>
                            </div>
                        </div>
                        <div class="text-[10px] font-bold text-blue-500 italic truncate">
                            ${item.note || '暂无备注'}
                        </div>
                    </div>
                `;
            }).join('');
            document.getElementById('cart-total').innerText = total.toFixed(2);
            if(!cart.length) {
                list.innerHTML = `<div class="h-full flex items-center justify-center text-slate-300 text-xs italic">购物车为空</div>`;
                selectedCartIndex = -1;
                syncNoteInput();
            }
        }

        function selectCartItem(idx) {
            selectedCartIndex = idx;
            renderCart();
            syncNoteInput();
        }

        function changeCount(idx, delta) {
            cart[idx].count += delta;
            if (cart[idx].count <= 0) {
                cart.splice(idx, 1);
                selectedCartIndex = -1;
            }
            renderCart();
            syncNoteInput();
        }

        // --- 备注增强逻辑 ---
        function renderTags() {
            const container = document.getElementById('tags-container');
            container.innerHTML = tags.map(t => `
                <button onclick="appendTag('${t}')" class="px-3 py-1.5 bg-white border border-slate-200 rounded-lg text-xs font-bold text-slate-600 hover:bg-blue-500 hover:text-white transition-all">
                    ${t}
                </button>
            `).join('');
        }

        function syncNoteInput() {
            const input = document.getElementById('manual-note');
            if (selectedCartIndex > -1) {
                input.value = cart[selectedCartIndex].note;
                input.disabled = false;
            } else {
                input.value = "";
                input.disabled = cart.length === 0;
            }
        }

        function updateNoteFromInput() {
            const val = document.getElementById('manual-note').value;
            if (selectedCartIndex > -1) {
                cart[selectedCartIndex].note = val;
                renderCart();
            }
        }

        function appendTag(tag) {
            if (selectedCartIndex === -1 && cart.length > 0) {
                toast("请先点击选择一个物品进行备注", "error");
                return;
            }
            const input = document.getElementById('manual-note');
            const currentVal = input.value;
            input.value = currentVal ? currentVal + " " + tag : tag;
            updateNoteFromInput();
        }

        function applyToAll() {
            const val = document.getElementById('manual-note').value;
            if (!cart.length) return;
            cart.forEach(item => item.note = val);
            renderCart();
            toast("备注已应用至全单");
        }

        function clearCart() { cart = []; selectedCartIndex = -1; renderCart(); syncNoteInput(); }

        function submitOrder() {
            if(!cart.length) return toast("购物车是空的", "error");
            const order = {
                id: Date.now(),
                num: String(DB.nextNum()).padStart(3, '0'),
                items: JSON.parse(JSON.stringify(cart)),
                total: parseFloat(document.getElementById('cart-total').innerText),
                status: "制作中",
                time: new Date().toLocaleTimeString('zh-CN', {hour12:false}),
                date: new Date().toISOString().split('T')[0]
            };
            orders.unshift(order);
            DB.set('orders', orders);
            clearCart();
            toast(`下单成功: #${order.num}`);
        }

        // --- 订单状态/后台逻辑 ---
        function renderActiveOrders() {
            const tbody = document.getElementById('active-orders-body');
            const active = orders.filter(o => o.status !== "已完成");
            tbody.innerHTML = active.map(o => `
                <tr class="fade-in">
                    <td class="p-4 font-black text-blue-600 text-xl italic">#${o.num}</td>
                    <td class="p-4">
                        <div class="text-[11px] font-bold text-slate-500">
                            ${o.items.map(i => `${i.name}x${i.count}${i.note?'('+i.note+')':''}`).join('、')}
                        </div>
                    </td>
                    <td class="p-4 text-right">
                        <div class="flex gap-2 justify-end">
                            ${STATUS_FLOW.map(s => `
                                <button onclick="updateStatus(${o.id}, '${s}')" class="px-4 py-2 rounded-xl text-xs font-bold transition-all ${o.status === s ? COLOR_MAP[s] : 'bg-slate-50 text-slate-300'}">
                                    ${s === "制作中" ? "制作" : (s === "请取餐" ? "叫号" : "完成")}
                                </button>
                            `).join('')}
                        </div>
                    </td>
                </tr>
            `).join('');
            if(!active.length) tbody.innerHTML = `<tr><td colspan="3" class="p-10 text-center text-slate-300 font-bold italic">暂无活跃订单</td></tr>`;
        }

        function updateStatus(oid, status) {
            const o = orders.find(x => x.id === oid);
            if(o) {
                o.status = status;
                DB.set('orders', orders);
                renderActiveOrders();
                toast(`#${o.num} 状态: ${status}`);
            }
        }

        function renderManageUI() {
            document.getElementById('product-manage-list').innerHTML = products.map(p => `
                <div class="flex justify-between items-center bg-slate-50 p-3 rounded-xl">
                    <span class="font-bold text-slate-700">${p.name} (¥${p.price})</span>
                    <button onclick="delProduct(${p.id})" class="text-red-400 font-bold text-xs">删除</button>
                </div>
            `).join('');
            document.getElementById('manage-tags-list').innerHTML = tags.map((t, idx) => `
                <div class="bg-slate-50 px-3 py-1 rounded-lg text-xs font-bold text-slate-600 flex items-center gap-2 border">
                    ${t} <button onclick="tags.splice(${idx},1); DB.set('tags', tags); renderManageUI()" class="text-red-400">×</button>
                </div>
            `).join('');
        }

        function addProduct() {
            const n = document.getElementById('m-name').value.trim();
            const p = parseFloat(document.getElementById('m-price').value);
            if(n && !isNaN(p)) {
                products.push({ id: Date.now(), name: n, price: p });
                DB.set('products', products);
                renderManageUI();
            }
        }

        function addTag() {
            const val = document.getElementById('tag-input').value.trim();
            if(val) {
                tags.push(val); DB.set('tags', tags);
                document.getElementById('tag-input').value = '';
                renderManageUI();
            }
        }

        function loadHistory() {
            const date = document.getElementById('history-date').value;
            const filtered = orders.filter(o => o.date === date);
            document.getElementById('history-body').innerHTML = filtered.map(o => `
                <tr class="hover:bg-slate-50">
                    <td class="p-4 font-black text-slate-400">#${o.num}</td>
                    <td class="p-4 text-[10px] font-bold text-slate-600">${o.items.map(i => i.name).join(', ')}</td>
                    <td class="p-4 text-right font-black italic">¥${o.total.toFixed(2)}</td>
                </tr>
            `).join('');
        }

        function toast(msg, type = "success") {
            const t = document.createElement('div');
            t.className = `px-6 py-3 mb-2 rounded-2xl shadow-xl font-black text-sm text-white ${type==='success'?'bg-slate-900':'bg-red-500'} fade-in`;
            t.innerText = msg;
            document.getElementById('toast-container').appendChild(t);
            setTimeout(() => t.remove(), 2500);
        }

        window.onload = () => {
            document.getElementById('history-date').value = new Date().toISOString().split('T')[0];
            switchTab('order');
        };
    </script>
</body>
</html>
