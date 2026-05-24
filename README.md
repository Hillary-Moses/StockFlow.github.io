[index.html](https://github.com/user-attachments/files/28187256/index.html)
# StockFlow.github.io
This is a stockflow page to run for stock updates.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=yes">
    <title>StockFlow App - Offline Inventory & Delivery</title>
    <!-- Tailwind CSS + Font Awesome Icons (simple, large buttons) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Firebase SDKs (modular, with offline persistence) -->
    <script type="importmap">
        {
            "imports": {
                "firebase/app": "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js",
                "firebase/firestore": "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js",
                "firebase/auth": "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js",
                "firebase/storage": "https://www.gstatic.com/firebasejs/10.8.0/firebase-storage.js"
            }
        }
    </script>
    <!-- SheetJS for CSV export -->
    <script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>
    <!-- html2pdf for PDF (simple) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js" integrity="sha512-GsLlZN/3F2ErC5ifS5QtgpiJtWd43JWSuIgh7mbzZ8zBps+dvLusV+eNQATqgA/HdeKFVgA5v3S/cIrLF7QnIg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
    <style>
        /* mobile-first large tap targets, custom scroll */
        .btn-large {
            min-height: 48px;
            font-size: 1rem;
            font-weight: 600;
        }
        input, select, textarea, button {
            font-size: 16px; /* prevents zoom on mobile */
        }
        .card-hover {
            transition: all 0.2s ease;
        }
        .low-stock-badge {
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { opacity: 0.7; }
            50% { opacity: 1; background-color: #dc2626; }
            100% { opacity: 0.7; }
        }
        .sync-toast {
            position: fixed;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: #1f2937;
            color: white;
            text-align: center;
            padding: 12px;
            border-radius: 40px;
            z-index: 50;
            font-size: 0.85rem;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body class="bg-gray-100 font-sans antialiased">

    <!-- Sync status toast -->
    <div id="syncToast" class="sync-toast hidden"></div>

    <!-- Main App Container -->
    <div id="app" class="max-w-7xl mx-auto px-3 py-4 pb-20">
        <!-- Login Screen -->
        <div id="loginSection" class="min-h-[70vh] flex items-center justify-center">
            <div class="bg-white rounded-2xl shadow-xl p-6 w-full max-w-md">
                <h2 class="text-3xl font-bold text-center text-blue-700 mb-6">StockFlow App</h2>
                <div class="space-y-4">
                    <input type="email" id="loginEmail" placeholder="Email" class="w-full p-3 border border-gray-300 rounded-xl text-lg">
                    <input type="password" id="loginPassword" placeholder="Password" class="w-full p-3 border border-gray-300 rounded-xl text-lg">
                    <button id="loginBtn" class="w-full bg-blue-600 text-white p-3 rounded-xl text-lg font-semibold btn-large">Login</button>
                    <p class="text-sm text-center text-gray-500 mt-2">Demo: admin@deliverstock.com / admin123 <br> staff@deliverstock.com / staff123</p>
                </div>
            </div>
        </div>

        <!-- Main App UI (hidden until auth) -->
        <div id="mainApp" class="hidden">
            <!-- Top bar: user + sync status -->
            <div class="flex justify-between items-center bg-white p-3 rounded-xl shadow mb-4">
                <h1 class="text-xl font-bold text-blue-800">📦 DeliverStock</h1>
                <div class="flex gap-2 items-center">
                    <span id="onlineStatus" class="text-xs bg-green-100 px-2 py-1 rounded-full">🟢 Online</span>
                    <span id="userRoleBadge" class="text-xs bg-gray-200 px-3 py-1 rounded-full font-mono"></span>
                    <button id="logoutBtn" class="bg-red-500 text-white px-4 py-1 rounded-lg text-sm">Logout</button>
                </div>
            </div>

            <!-- Tab Navigation (large buttons) -->
            <div class="grid grid-cols-4 gap-2 mb-6 bg-white p-2 rounded-xl shadow">
                <button data-tab="dashboard" class="tab-btn py-3 rounded-xl text-base font-semibold bg-blue-600 text-white">📊 Dashboard</button>
                <button data-tab="products" class="tab-btn py-3 rounded-xl text-base font-semibold bg-gray-200 text-gray-700">📦 Stock</button>
                <button data-tab="deliveries" class="tab-btn py-3 rounded-xl text-base font-semibold bg-gray-200 text-gray-700">🚚 Deliveries</button>
                <button data-tab="admin" id="adminTabBtn" class="tab-btn py-3 rounded-xl text-base font-semibold bg-gray-200 text-gray-700 hidden">⚙️ Admin</button>
            </div>

            <!-- Dashboard Tab -->
            <div id="dashboardTab" class="tab-content">
                <div class="grid grid-cols-2 gap-3 mb-5">
                    <div class="bg-white rounded-2xl p-4 shadow text-center">
                        <p class="text-gray-500 text-sm">📦 Total Products</p>
                        <p id="totalProductsCount" class="text-3xl font-bold">0</p>
                    </div>
                    <div class="bg-white rounded-2xl p-4 shadow text-center">
                        <p class="text-gray-500 text-sm">⚠️ Low Stock</p>
                        <p id="lowStockCount" class="text-3xl font-bold text-red-600">0</p>
                    </div>
                    <div class="bg-white rounded-2xl p-4 shadow text-center">
                        <p class="text-gray-500 text-sm">✅ Today's Deliveries</p>
                        <p id="todayDeliveries" class="text-3xl font-bold">0</p>
                    </div>
                    <div class="bg-white rounded-2xl p-4 shadow text-center">
                        <p class="text-gray-500 text-sm">🚚 Pending Outgoing</p>
                        <p id="pendingOutgoing" class="text-3xl font-bold">0</p>
                    </div>
                </div>
                <div class="bg-white rounded-2xl p-4 shadow">
                    <h3 class="font-bold text-lg mb-2">⚠️ Low Stock Alerts</h3>
                    <div id="lowStockList" class="space-y-2 max-h-60 overflow-y-auto"></div>
                </div>
            </div>

            <!-- Products Tab (Stock Management) -->
            <div id="productsTab" class="tab-content hidden">
                <div class="flex flex-wrap gap-2 mb-4">
                    <button id="addProductBtn" class="bg-green-600 text-white px-5 py-2 rounded-xl flex-1 btn-large">➕ New Product</button>
                    <input type="text" id="productSearch" placeholder="🔍 Search by name or SKU" class="flex-1 border p-2 rounded-xl">
                    <button id="exportProductsCsv" class="bg-gray-700 text-white px-4 rounded-xl">📎 CSV</button>
                </div>
                <div id="productsList" class="space-y-3 max-h-[65vh] overflow-y-auto"></div>
            </div>

            <!-- Deliveries Tab (Stock-in + Outgoing Delivery) -->
            <div id="deliveriesTab" class="tab-content hidden">
                <div class="grid grid-cols-2 gap-2 mb-4">
                    <button id="quickStockInBtn" class="bg-emerald-600 text-white p-3 rounded-xl btn-large">📥 + Stock-In</button>
                    <button id="quickDeliveryBtn" class="bg-amber-600 text-white p-3 rounded-xl btn-large">🚚 New Delivery</button>
                </div>
                <div class="bg-white rounded-xl p-3 mb-3 flex gap-2">
                    <input type="text" id="deliverySearch" placeholder="Search deliveries (product/driver)" class="flex-1 border p-2 rounded-lg">
                    <select id="filterDeliveryStatus" class="border rounded-lg p-2">
                        <option value="all">All Status</option>
                        <option value="pending">Pending</option>
                        <option value="delivered">Delivered</option>
                        <option value="failed">Failed</option>
                    </select>
                </div>
                <div id="deliveriesList" class="space-y-3 max-h-[60vh] overflow-y-auto"></div>
            </div>

            <!-- Admin Tab (Manage Drivers & Users) -->
            <div id="adminTab" class="tab-content hidden">
                <div class="bg-white rounded-xl p-4 mb-4">
                    <div class="flex justify-between items-center"><h3 class="font-bold text-lg">👨‍✈️ Drivers</h3><button id="addDriverBtn" class="bg-blue-500 text-white px-3 py-1 rounded-lg">+ Add Driver</button></div>
                    <div id="driversList" class="mt-2 space-y-2"></div>
                </div>
                <div class="bg-white rounded-xl p-4">
                    <h3 class="font-bold text-lg">👥 Staff Management (Admin only)</h3>
                    <div class="flex gap-2 mt-2"><input id="newStaffEmail" placeholder="Email" class="border p-2 rounded flex-1"><input id="newStaffPassword" placeholder="Password" class="border p-2 rounded"><button id="createStaffBtn" class="bg-indigo-600 text-white px-3 rounded">Create Staff</button></div>
                    <div id="staffList" class="mt-3 space-y-2 text-sm"></div>
                </div>
                <div class="bg-white rounded-xl p-4 mt-4"><button id="exportAllReportsBtn" class="bg-purple-700 text-white w-full py-2 rounded-xl">📄 Export Full Report (Stock+Deliveries CSV)</button></div>
            </div>
        </div>
    </div>

    <!-- Modals (Product/Delivery/Driver forms) -->
    <div id="productModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-2xl max-w-md w-full p-5"><h3 class="text-xl font-bold mb-3" id="modalTitle">Add Product</h3>
            <input id="prodName" placeholder="Product Name" class="w-full border p-2 rounded mb-2"><input id="prodSku" placeholder="SKU (optional)" class="w-full border p-2 rounded mb-2"><input id="prodStock" type="number" placeholder="Current Stock" class="w-full border p-2 rounded mb-2"><input id="prodThreshold" type="number" placeholder="Low stock threshold" class="w-full border p-2 rounded mb-2"><input id="prodBarcode" placeholder="Barcode (optional)" class="w-full border p-2 rounded mb-2">
            <div class="flex gap-2"><button id="saveProductBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button><button id="closeProductModal" class="bg-gray-400 px-4 py-2 rounded">Cancel</button></div>
        </div>
    </div>

    <div id="deliveryModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-2xl max-w-md w-full p-5"><h3 class="text-xl font-bold mb-2" id="deliveryModalTitle">New Stock-In</h3>
            <select id="deliveryType" class="w-full border p-2 rounded mb-2"><option value="in">📥 Stock-In (Increase stock)</option><option value="out">🚚 Outgoing Delivery</option></select>
            <select id="deliveryProductId" class="w-full border p-2 rounded mb-2"><option>Select Product</option></select>
            <input id="deliveryQuantity" type="number" placeholder="Quantity" class="w-full border p-2 rounded mb-2">
            <div id="outgoingFields" class="hidden">
                <input id="customerName" placeholder="Customer name" class="w-full border p-2 rounded mb-2"><input id="customerAddress" placeholder="Address" class="w-full border p-2 rounded mb-2">
                <select id="driverId" class="w-full border p-2 rounded mb-2"><option value="">Assign Driver</option></select>
                <select id="deliveryStatus" class="w-full border p-2 rounded mb-2"><option value="pending">Pending</option><option value="delivered">Delivered</option><option value="failed">Failed</option></select>
                <label class="block text-sm">📸 Proof Photo (optional)</label><input type="file" id="proofPhoto" accept="image/*" class="border p-1 rounded w-full mb-2">
            </div>
            <div class="flex gap-2"><button id="saveDeliveryBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button><button id="closeDeliveryModal" class="bg-gray-400 px-4 py-2 rounded">Cancel</button></div>
        </div>
    </div>

    <div id="driverModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50 p-4"><div class="bg-white rounded-2xl p-5 w-80"><h3 class="font-bold">Add Driver</h3><input id="driverName" placeholder="Driver name" class="border p-2 w-full my-2 rounded"><div class="flex gap-2"><button id="confirmDriverBtn" class="bg-green-600 text-white px-4 py-2 rounded">Save</button><button id="closeDriverModal" class="bg-gray-400 px-4 py-2 rounded">Cancel</button></div></div></div>

    <script type="module">
        import { initializeApp } from "firebase/app";
        import { getFirestore, collection, doc, getDocs, addDoc, updateDoc, deleteDoc, query, where, onSnapshot, enableIndexedDbPersistence, Timestamp, increment, orderBy, getDoc, writeBatch } from "firebase/firestore";
        import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword, signOut, onAuthStateChanged } from "firebase/auth";
        import { getStorage, ref, uploadBytes, getDownloadURL } from "firebase/storage";

        // ==================== FIREBASE CONFIG (Replace with your own) ====================
        const firebaseConfig = {
            apiKey: "AIzaSyDummyKeyReplaceWithYourOwn",   // <-- IMPORTANT: Replace with actual Firebase config
            authDomain: "deliverstock-demo.firebaseapp.com",
            projectId: "deliverstock-demo",
            storageBucket: "deliverstock-demo.appspot.com",
            messagingSenderId: "123456789012",
            appId: "1:123456789012:web:abcdef123456"
        };
        // NOTE: For demo to fully work offline, you must create a Firebase project, enable Email/Password auth, Firestore, Storage.
        // This code is fully functional after configuration. For testing, demo shows concept with offline persistence.

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const storage = getStorage(app);

        // Enable offline persistence (critical for offline-first)
        enableIndexedDbPersistence(db).catch((err) => { if (err.code === 'failed-precondition') console.warn("multiple tabs"); else console.error(err) });

        // Global state
        let currentUser = null;
        let userRole = "staff"; // admin / staff
        let productsCache = [];
        let deliveriesCache = [];
        let driversCache = [];

        // DOM elements
        const loginSection = document.getElementById('loginSection');
        const mainApp = document.getElementById('mainApp');
        const syncToast = document.getElementById('syncToast');
        const onlineSpan = document.getElementById('onlineStatus');
        const userRoleBadge = document.getElementById('userRoleBadge');

        function showSyncMsg(msg, isError = false) {
            syncToast.innerText = msg;
            syncToast.classList.remove('hidden');
            setTimeout(() => syncToast.classList.add('hidden'), 3000);
        }

        window.addEventListener('online', () => { onlineSpan.innerText = '🟢 Online'; showSyncMsg('Connected - Syncing automatically'); });
        window.addEventListener('offline', () => { onlineSpan.innerText = '🔴 Offline'; showSyncMsg('Offline mode - changes saved locally'); });

        // Helper: check admin role via custom claim / firestore user doc
        async function setUserRole(user) {
            if (!user) return;
            const userDoc = await getDoc(doc(db, "users", user.uid)).catch(()=>null);
            if (userDoc && userDoc.exists()) userRole = userDoc.data().role;
            else userRole = "staff";
            if (user.email === "admin@deliverstock.com") userRole = "admin"; // fallback demo
            userRoleBadge.innerText = userRole.toUpperCase();
            document.getElementById('adminTabBtn').classList.toggle('hidden', userRole !== 'admin');
            if (userRole === 'admin') document.querySelector('[data-tab="admin"]').classList.remove('hidden');
            else document.querySelector('[data-tab="admin"]')?.classList.add('hidden');
        }

        // Real-time listeners with offline sync
        function subscribeProducts() {
            const q = query(collection(db, "products"), orderBy("name"));
            onSnapshot(q, (snap) => {
                productsCache = snap.docs.map(d => ({ id: d.id, ...d.data() }));
                renderProducts();
                updateDashboard();
                updateDeliveryProductSelects();
            }, (err) => console.warn(err));
        }

        function subscribeDeliveries() {
            const q = query(collection(db, "deliveries"), orderBy("timestamp", "desc"));
            onSnapshot(q, (snap) => {
                deliveriesCache = snap.docs.map(d => ({ id: d.id, ...d.data() }));
                renderDeliveries();
                updateDashboard();
            });
        }

        function subscribeDrivers() {
            onSnapshot(collection(db, "drivers"), (snap) => {
                driversCache = snap.docs.map(d => ({ id: d.id, name: d.data().name }));
                renderDriversList();
                updateDriverSelect();
            });
        }

        // Dashboard update
        function updateDashboard() {
            document.getElementById('totalProductsCount').innerText = productsCache.length;
            const lowStock = productsCache.filter(p => p.currentStock <= p.lowThreshold);
            document.getElementById('lowStockCount').innerText = lowStock.length;
            const today = new Date().toDateString();
            const todayDeliveries = deliveriesCache.filter(d => d.timestamp && new Date(d.timestamp.toDate?.() || d.timestamp).toDateString() === today && d.type === 'out');
            document.getElementById('todayDeliveries').innerText = todayDeliveries.length;
            const pendingOut = deliveriesCache.filter(d => d.type === 'out' && d.status === 'pending').length;
            document.getElementById('pendingOutgoing').innerText = pendingOut;
            // low stock list
            const container = document.getElementById('lowStockList');
            if (lowStock.length === 0) container.innerHTML = '<div class="text-gray-500">✅ All stock levels healthy</div>';
            else container.innerHTML = lowStock.map(p => `<div class="bg-red-50 p-2 rounded flex justify-between"><span>⚠️ ${p.name}</span><span class="font-bold">Stock: ${p.currentStock} / ${p.lowThreshold}</span></div>`).join('');
        }

        // Product rendering with search
        function renderProducts() {
            const search = document.getElementById('productSearch')?.value.toLowerCase() || '';
            let filtered = productsCache.filter(p => p.name.toLowerCase().includes(search) || (p.sku || '').toLowerCase().includes(search));
            const container = document.getElementById('productsList');
            if (!container) return;
            if (filtered.length === 0) { container.innerHTML = '<div class="text-center p-4">No products</div>'; return; }
            container.innerHTML = filtered.map(p => `
                <div class="bg-white p-4 rounded-xl shadow flex flex-wrap justify-between items-center">
                    <div><span class="font-bold text-lg">${p.name}</span> <span class="text-xs text-gray-500">SKU:${p.sku || '-'}</span><br/>
                    <span class="text-2xl font-mono">📦 ${p.currentStock}</span> ${p.currentStock <= p.lowThreshold ? '<span class="bg-red-500 text-white text-xs px-2 py-1 rounded-full low-stock-badge">Low Stock</span>' : ''}
                    <div class="text-sm">Threshold: ${p.lowThreshold}</div></div>
                    <div class="flex gap-2 mt-2">${userRole === 'admin' ? `<button class="edit-product bg-blue-500 text-white px-3 py-1 rounded" data-id="${p.id}">✏️ Edit</button><button class="delete-product bg-red-500 text-white px-3 py-1 rounded" data-id="${p.id}">🗑️</button>` : ''}
                    <button class="quick-stockin bg-emerald-600 text-white px-3 py-1 rounded" data-id="${p.id}" data-name="${p.name}">➕ Stock-In</button></div>
                </div>`).join('');
            document.querySelectorAll('.edit-product').forEach(btn => btn.addEventListener('click', (e) => openEditProduct(btn.dataset.id)));
            document.querySelectorAll('.delete-product').forEach(btn => btn.addEventListener('click', async (e) => { if(confirm('Delete product?')) await deleteDoc(doc(db,"products",btn.dataset.id)); }));
            document.querySelectorAll('.quick-stockin').forEach(btn => btn.addEventListener('click', () => openQuickStockIn(btn.dataset.id)));
        }

        function openEditProduct(id) { const p = productsCache.find(x=>x.id===id); if(p){ document.getElementById('prodName').value=p.name; document.getElementById('prodSku').value=p.sku||''; document.getElementById('prodStock').value=p.currentStock; document.getElementById('prodThreshold').value=p.lowThreshold; document.getElementById('prodBarcode').value=p.barcode||''; document.getElementById('modalTitle').innerText='Edit Product'; window._editId=id; document.getElementById('productModal').classList.remove('hidden'); } }

        async function saveProduct() { const name=document.getElementById('prodName').value; const sku=document.getElementById('prodSku').value; const stock=parseInt(document.getElementById('prodStock').value); const thresh=parseInt(document.getElementById('prodThreshold').value); const barcode=document.getElementById('prodBarcode').value; if(window._editId){ await updateDoc(doc(db,"products",window._editId),{name,sku,currentStock:stock,lowThreshold:thresh,barcode}); } else { await addDoc(collection(db,"products"),{name,sku,currentStock:stock,lowThreshold:thresh,barcode}); } closeProductModal(); renderProducts(); }

        function closeProductModal() { document.getElementById('productModal').classList.add('hidden'); window._editId=null; document.getElementById('prodName').value=''; }

        function updateDeliveryProductSelects() { const select = document.getElementById('deliveryProductId'); if(select) select.innerHTML = productsCache.map(p => `<option value="${p.id}">${p.name} (Stock: ${p.currentStock})</option>`).join(''); }
        function updateDriverSelect() { const driverSelect = document.getElementById('driverId'); if(driverSelect) driverSelect.innerHTML = '<option value="">No Driver</option>' + driversCache.map(d => `<option value="${d.id}">${d.name}</option>`).join(''); }

        // Delivery: Stock-In & Outgoing with atomic increment
        async function saveDelivery() {
            const type = document.getElementById('deliveryType').value;
            const productId = document.getElementById('deliveryProductId').value;
            const qty = parseInt(document.getElementById('deliveryQuantity').value);
            if (!productId || isNaN(qty) || qty <= 0) { alert("Valid product & quantity required"); return; }
            const productRef = doc(db, "products", productId);
            if (type === 'in') {
                await updateDoc(productRef, { currentStock: increment(qty) });
                await addDoc(collection(db,"deliveries"),{ type:"in", productId, quantity:qty, timestamp: Timestamp.now(), note: "Stock-in" });
                showSyncMsg("Stock added (syncs offline)");
            } else { // outgoing
                const custName = document.getElementById('customerName').value;
                const address = document.getElementById('customerAddress').value;
                const driverIdVal = document.getElementById('driverId').value || null;
                const status = document.getElementById('deliveryStatus').value;
                // photo handling base64 (simple offline proof)
                let photoURL = "";
                const photoFile = document.getElementById('proofPhoto').files[0];
                if (photoFile) { const reader = await new Promise((res) => { const fr = new FileReader(); fr.onload = () => res(fr.result); fr.readAsDataURL(photoFile); }); photoURL = reader; }
                // reduce stock on outgoing delivery
                await updateDoc(productRef, { currentStock: increment(-qty) });
                await addDoc(collection(db,"deliveries"),{ type:"out", productId, quantity:qty, customerName:custName, address, driverId:driverIdVal, status, photoProof:photoURL, timestamp: Timestamp.now() });
                showSyncMsg("Outgoing delivery logged");
            }
            document.getElementById('deliveryModal').classList.add('hidden');
            document.getElementById('deliveryQuantity').value = "";
            document.getElementById('proofPhoto').value = "";
        }

        function openQuickStockIn(prodId) { document.getElementById('deliveryModalTitle').innerText = "Stock-In"; document.getElementById('deliveryType').value = "in"; document.getElementById('deliveryProductId').value = prodId; document.getElementById('outgoingFields').classList.add('hidden'); document.getElementById('deliveryModal').classList.remove('hidden'); }

        function renderDeliveries() {
            const search = document.getElementById('deliverySearch')?.value.toLowerCase() || '';
            const statusFilter = document.getElementById('filterDeliveryStatus')?.value || 'all';
            let filtered = deliveriesCache.filter(d => (d.productId && productsCache.find(p=>p.id===d.productId)?.name.toLowerCase().includes(search)) || (d.customerName||'').toLowerCase().includes(search));
            if(statusFilter !== 'all') filtered = filtered.filter(d => d.status === statusFilter);
            const container = document.getElementById('deliveriesList');
            if(!container) return;
            if(filtered.length===0) { container.innerHTML='<div class="bg-white p-4 rounded-xl text-center">No deliveries</div>'; return; }
            container.innerHTML = filtered.map(d => {
                const prod = productsCache.find(p=>p.id===d.productId);
                const driver = driversCache.find(dr=>dr.id===d.driverId);
                const dateStr = d.timestamp?.toDate?.() ? d.timestamp.toDate().toLocaleString() : new Date(d.timestamp).toLocaleString();
                return `<div class="bg-white p-4 rounded-xl shadow"><div class="flex justify-between"><span class="font-bold">${d.type === 'in' ? '📥 Stock-In' : '🚚 Delivery'}</span><span class="text-xs">${dateStr}</span></div><div>Product: ${prod?.name || d.productId} | Qty: ${d.quantity}</div>${d.customerName ? `<div>👤 ${d.customerName} | ${d.address||''}</div>` : ''}${d.driverId ? `<div>Driver: ${driver?.name || '—'}</div>` : ''}<div class="mt-2 flex gap-2 items-center">${d.type === 'out' ? `<select data-id="${d.id}" class="status-change border rounded p-1 text-sm"><option ${d.status==='pending'?'selected':''}>pending</option><option ${d.status==='delivered'?'selected':''}>delivered</option><option ${d.status==='failed'?'selected':''}>failed</option></select>` : `<span class="bg-gray-200 px-2 py-1 rounded text-xs">${d.status||'Completed'}</span>`} ${d.photoProof ? `<span class="text-xs text-blue-500">📸 Proof</span>` : ''}</div></div>`;
            }).join('');
            document.querySelectorAll('.status-change').forEach(select => select.addEventListener('change', async (e) => {
                const newStatus = e.target.value; const deliveryId = e.target.dataset.id;
                const delivery = deliveriesCache.find(d=>d.id===deliveryId);
                if(!delivery) return;
                if(newStatus === 'failed' && delivery.status !== 'failed') {
                    // revert stock: increment back quantity
                    const prodRef = doc(db,"products",delivery.productId);
                    await updateDoc(prodRef, { currentStock: increment(delivery.quantity) });
                } else if(newStatus === 'delivered' && delivery.status !== 'delivered' && delivery.status !== 'failed') { }
                else if(newStatus === 'pending' && delivery.status === 'failed') { const prodRef = doc(db,"products",delivery.productId); await updateDoc(prodRef, { currentStock: increment(-delivery.quantity) }); }
                await updateDoc(doc(db,"deliveries",deliveryId), { status: newStatus });
            }));
        }

        // Admin drivers & staff
        async function addDriver() { const name=document.getElementById('driverName').value; if(name) await addDoc(collection(db,"drivers"),{name}); document.getElementById('driverModal').classList.add('hidden'); }
        function renderDriversList() { const container=document.getElementById('driversList'); if(container) container.innerHTML = driversCache.map(d => `<div class="flex justify-between p-2 bg-gray-50 rounded"><span>${d.name}</span><button class="delete-driver text-red-500" data-id="${d.id}">Delete</button></div>`).join(''); document.querySelectorAll('.delete-driver').forEach(btn => btn.addEventListener('click', async (e) => { if(confirm('Delete driver?')) await deleteDoc(doc(db,"drivers",btn.dataset.id)); })); }

        // CSV Export
        async function exportFullReports() { let csvRows = [["Type","Product","Quantity","Customer","Driver","Status","Date"]]; deliveriesCache.forEach(d => { const prod = productsCache.find(p=>p.id===d.productId); csvRows.push([d.type, prod?.name||'', d.quantity, d.customerName||'', d.driverId||'', d.status||'done', d.timestamp?.toDate?.()||'']); }); const ws = XLSX.utils.aoa_to_sheet(csvRows); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb, ws, "Deliveries"); XLSX.writeFile(wb, `inventory_report_${new Date().toISOString()}.xlsx`); }

        // Auth & UI Switching
        document.getElementById('loginBtn').onclick = async () => { const email=document.getElementById('loginEmail').value; const pwd=document.getElementById('loginPassword').value; try{ await signInWithEmailAndPassword(auth,email,pwd); }catch(e){ alert("Login failed: "+e.message); } };
        document.getElementById('logoutBtn').onclick = async () => { await signOut(auth); };
        onAuthStateChanged(auth, async (user) => { if(user){ currentUser=user; await setUserRole(user); loginSection.classList.add('hidden'); mainApp.classList.remove('hidden'); subscribeProducts(); subscribeDeliveries(); subscribeDrivers(); } else { loginSection.classList.remove('hidden'); mainApp.classList.add('hidden'); } });

        // tab switching
        document.querySelectorAll('.tab-btn').forEach(btn => btn.addEventListener('click', () => { const tab=btn.dataset.tab; document.querySelectorAll('.tab-content').forEach(t => t.classList.add('hidden')); document.getElementById(`${tab}Tab`).classList.remove('hidden'); }));
        document.getElementById('addProductBtn').onclick = () => { document.getElementById('modalTitle').innerText='Add Product'; window._editId=null; document.getElementById('prodName').value=''; document.getElementById('prodSku').value=''; document.getElementById('prodStock').value=0; document.getElementById('prodThreshold').value=5; document.getElementById('productModal').classList.remove('hidden'); };
        document.getElementById('saveProductBtn').onclick = saveProduct;
        document.getElementById('closeProductModal').onclick = closeProductModal;
        document.getElementById('quickStockInBtn').onclick = () => { document.getElementById('deliveryModalTitle').innerText='Stock-In'; document.getElementById('deliveryType').value='in'; document.getElementById('outgoingFields').classList.add('hidden'); document.getElementById('deliveryModal').classList.remove('hidden'); };
        document.getElementById('quickDeliveryBtn').onclick = () => { document.getElementById('deliveryModalTitle').innerText='Outgoing Delivery'; document.getElementById('deliveryType').value='out'; document.getElementById('outgoingFields').classList.remove('hidden'); document.getElementById('deliveryModal').classList.remove('hidden'); };
        document.getElementById('saveDeliveryBtn').onclick = saveDelivery;
        document.getElementById('closeDeliveryModal').onclick = () => document.getElementById('deliveryModal').classList.add('hidden');
        document.getElementById('addDriverBtn').onclick = () => document.getElementById('driverModal').classList.remove('hidden');
        document.getElementById('confirmDriverBtn').onclick = addDriver;
        document.getElementById('closeDriverModal').onclick = () => document.getElementById('driverModal').classList.add('hidden');
        document.getElementById('exportProductsCsv').onclick = () => { const ws = XLSX.utils.json_to_sheet(productsCache.map(p=>({Name:p.name,SKU:p.sku,Stock:p.currentStock,Threshold:p.lowThreshold}))); const wb=XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb,ws,"Products"); XLSX.writeFile(wb,"products_export.xlsx"); };
        document.getElementById('exportAllReportsBtn').onclick = exportFullReports;
        document.getElementById('createStaffBtn').onclick = async () => { if(userRole !== 'admin') return alert("Admin only"); const email=document.getElementById('newStaffEmail').value; const pwd=document.getElementById('newStaffPassword').value; try{ const cred = await createUserWithEmailAndPassword(auth,email,pwd); await setDoc(doc(db,"users",cred.user.uid),{ role:"staff", email }); alert("Staff created"); }catch(e){ alert(e.message); } };
        // helper setDoc import
        import { setDoc } from "firebase/firestore";
        window.setDoc = setDoc;
        // on load demo users hint
        console.log("Offline-first ready. Ensure Firebase config is replaced for full sync.");
    </script>
</body>
</html>
