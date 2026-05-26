<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StockFlow - Multi-Location Inventory</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>
    <style>
        .btn-large {
            min-height: 48px;
            font-size: 1rem;
            font-weight: 600;
        }
        input, select, textarea, button {
            font-size: 16px;
        }
        .low-stock-badge {
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { opacity: 0.7; }
            50% { opacity: 1; background-color: #dc2626; }
            100% { opacity: 0.7; }
        }
    </style>
</head>
<body class="bg-gray-100 font-sans antialiased">

    <!-- Main App Container -->
    <div id="app" class="max-w-7xl mx-auto px-3 py-4 pb-20">
        
        <!-- Top bar -->
        <div class="flex justify-between items-center bg-white p-4 rounded-xl shadow mb-4">
            <h1 class="text-2xl font-bold text-blue-800">📦 StockFlow Multi-Location</h1>
            <div class="flex gap-2 items-center">
                <span id="totalItems" class="text-sm bg-blue-100 px-3 py-1 rounded-full">Items: 0</span>
                <button id="clearAllBtn" class="bg-red-500 text-white px-3 py-1 rounded text-sm">Clear All</button>
            </div>
        </div>

        <!-- Tab Navigation -->
        <div class="grid grid-cols-5 gap-2 mb-6 bg-white p-2 rounded-xl shadow">
            <button data-tab="dashboard" class="tab-btn py-3 rounded-xl font-semibold bg-blue-600 text-white">📊 Dashboard</button>
            <button data-tab="locations" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700">📍 Locations</button>
            <button data-tab="products" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700">📦 Products</button>
            <button data-tab="transfers" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700">🚚 Transfers</button>
            <button data-tab="deliveries" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700">📤 Deliveries</button>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboardTab" class="tab-content">
            <div class="grid grid-cols-2 gap-3 mb-5">
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">📦 Total Products</p>
                    <p id="totalProducts" class="text-3xl font-bold">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">📍 Total Locations</p>
                    <p id="totalLocations" class="text-3xl font-bold">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">📤 Total Deliveries</p>
                    <p id="totalDeliveries" class="text-3xl font-bold">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">⚠️ Low Stock Alerts</p>
                    <p id="lowStockAlert" class="text-3xl font-bold text-red-600">0</p>
                </div>
            </div>
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
                <div class="bg-white rounded-xl p-4 shadow">
                    <h3 class="font-bold text-lg mb-3">📍 Stock by Location</h3>
                    <div id="locationStockList" class="space-y-2"></div>
                </div>
                <div class="bg-white rounded-xl p-4 shadow">
                    <h3 class="font-bold text-lg mb-3">⚠️ Low Stock Alerts</h3>
                    <div id="lowStockList" class="space-y-2"></div>
                </div>
            </div>
        </div>

        <!-- Locations Tab -->
        <div id="locationsTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="addLocationBtn" class="bg-green-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">➕ Add Location</button>
                <input type="text" id="searchLocation" placeholder="🔍 Search locations..." class="flex-1 border p-2 rounded-xl">
            </div>
            <div id="locationsList" class="space-y-3"></div>
        </div>

        <!-- Products Tab -->
        <div id="productsTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="addProductBtn" class="bg-green-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">➕ Add Product</button>
                <input type="text" id="searchProduct" placeholder="🔍 Search products..." class="flex-1 border p-2 rounded-xl">
                <button id="exportProductsBtn" class="bg-gray-700 text-white px-4 rounded-xl">📥 Export</button>
            </div>
            <div id="productsList" class="space-y-3"></div>
        </div>

        <!-- Transfers Tab -->
        <div id="transfersTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="addTransferBtn" class="bg-amber-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">🚚 New Transfer</button>
                <input type="text" id="searchTransfer" placeholder="🔍 Search transfers..." class="flex-1 border p-2 rounded-xl">
            </div>
            <div id="transfersList" class="space-y-3"></div>
        </div>

        <!-- Deliveries Tab -->
        <div id="deliveriesTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="addDeliveryBtn" class="bg-blue-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">📤 Record Delivery</button>
                <input type="text" id="searchDelivery" placeholder="🔍 Search deliveries..." class="flex-1 border p-2 rounded-xl">
            </div>
            <div id="deliveriesList" class="space-y-3"></div>
        </div>

    </div>

    <!-- Location Modal -->
    <div id="locationModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4" id="locationModalTitle">Add Location</h3>
            <input id="locationName" placeholder="Location Name (e.g., Ibadan HQ)" class="w-full border p-2 rounded mb-2">
            <input id="locationCity" placeholder="City" class="w-full border p-2 rounded mb-2">
            <input id="locationState" placeholder="State" class="w-full border p-2 rounded mb-2">
            <input id="locationAddress" placeholder="Address" class="w-full border p-2 rounded mb-4">
            <div class="flex gap-2">
                <button id="saveLocationBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button>
                <button id="closeLocationModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Product Modal -->
    <div id="productModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4" id="productModalTitle">Add Product</h3>
            <input id="productName" placeholder="Product Name" class="w-full border p-2 rounded mb-2">
            <input id="productSku" placeholder="SKU" class="w-full border p-2 rounded mb-2">
            <input id="productPrice" type="number" placeholder="Price" class="w-full border p-2 rounded mb-2">
            <input id="productThreshold" type="number" placeholder="Low Stock Threshold" class="w-full border p-2 rounded mb-4">
            <div class="flex gap-2">
                <button id="saveProductBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button>
                <button id="closeProductModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Stock Modal -->
    <div id="stockModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4">Add Stock</h3>
            <select id="stockLocation" class="w-full border p-2 rounded mb-2">
                <option>Select Location</option>
            </select>
            <input id="stockQuantity" type="number" placeholder="Quantity" class="w-full border p-2 rounded mb-4">
            <div class="flex gap-2">
                <button id="saveStockBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Add Stock</button>
                <button id="closeStockModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Transfer Modal -->
    <div id="transferModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4">Transfer Stock</h3>
            <select id="transferProduct" class="w-full border p-2 rounded mb-2">
                <option>Select Product</option>
            </select>
            <select id="transferFrom" class="w-full border p-2 rounded mb-2">
                <option>From Location</option>
            </select>
            <select id="transferTo" class="w-full border p-2 rounded mb-2">
                <option>To Location</option>
            </select>
            <input id="transferQuantity" type="number" placeholder="Quantity" class="w-full border p-2 rounded mb-4">
            <div class="flex gap-2">
                <button id="saveTransferBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Transfer</button>
                <button id="closeTransferModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Delivery Modal -->
    <div id="deliveryModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4" id="deliveryModalTitle">Record Delivery</h3>
            <select id="deliveryLocation" class="w-full border p-2 rounded mb-2">
                <option>Select Location</option>
            </select>
            <select id="deliveryProduct" class="w-full border p-2 rounded mb-2">
                <option>Select Product</option>
            </select>
            <input id="deliveryQuantityInStock" type="number" placeholder="Quantity in Stock" class="w-full border p-2 rounded mb-2" readonly>
            <input id="deliveryQuantityDelivered" type="number" placeholder="Quantity Delivered" class="w-full border p-2 rounded mb-2">
            <input id="deliveryCustomer" placeholder="Customer Name" class="w-full border p-2 rounded mb-2">
            <input id="deliveryAddress" placeholder="Delivery Address" class="w-full border p-2 rounded mb-2">
            <select id="deliveryStatus" class="w-full border p-2 rounded mb-4">
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
                <option value="failed">Failed</option>
            </select>
            <div class="flex gap-2">
                <button id="saveDeliveryBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button>
                <button id="closeDeliveryModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <script>
        // ============ LOCAL STORAGE DATA ============
        let locations = JSON.parse(localStorage.getItem('stockflow_locations')) || [];
        let products = JSON.parse(localStorage.getItem('stockflow_products')) || [];
        let stock = JSON.parse(localStorage.getItem('stockflow_stock')) || [];
        let transfers = JSON.parse(localStorage.getItem('stockflow_transfers')) || [];
        let deliveries = JSON.parse(localStorage.getItem('stockflow_deliveries')) || [];

        let editingLocationId = null;
        let editingProductId = null;
        let editingDeliveryId = null;
        let currentStockProductId = null;

        // ============ SAVE TO LOCAL STORAGE ============
        function saveData() {
            localStorage.setItem('stockflow_locations', JSON.stringify(locations));
            localStorage.setItem('stockflow_products', JSON.stringify(products));
            localStorage.setItem('stockflow_stock', JSON.stringify(stock));
            localStorage.setItem('stockflow_transfers', JSON.stringify(transfers));
            localStorage.setItem('stockflow_deliveries', JSON.stringify(deliveries));
            updateDashboard();
            renderLocations();
            renderProducts();
            renderTransfers();
            renderDeliveries();
        }

        // ============ DASHBOARD ============
        function updateDashboard() {
            document.getElementById('totalProducts').innerText = products.length;
            document.getElementById('totalLocations').innerText = locations.length;
            document.getElementById('totalDeliveries').innerText = deliveries.length;
            const lowStock = products.filter(p => {
                const totalStock = stock.filter(s => s.productId === p.id).reduce((sum, s) => sum + s.quantity, 0);
                return totalStock <= p.threshold;
            });
            document.getElementById('lowStockAlert').innerText = lowStock.length;

            // Location stock breakdown
            const locStockContainer = document.getElementById('locationStockList');
            if (locations.length === 0) {
                locStockContainer.innerHTML = '<div class="text-gray-500">No locations yet</div>';
            } else {
                locStockContainer.innerHTML = locations.map(loc => {
                    const locStock = stock.filter(s => s.locationId === loc.id).reduce((sum, s) => sum + s.quantity, 0);
                    return `<div class="bg-blue-50 p-2 rounded flex justify-between"><span>📍 ${loc.name}</span><span class="font-bold">${locStock} units</span></div>`;
                }).join('');
            }

            // Low stock list
            const lowStockContainer = document.getElementById('lowStockList');
            if (lowStock.length === 0) {
                lowStockContainer.innerHTML = '<div class="text-gray-500">✅ All stock levels healthy</div>';
            } else {
                lowStockContainer.innerHTML = lowStock.map(p => {
                    const totalStock = stock.filter(s => s.productId === p.id).reduce((sum, s) => sum + s.quantity, 0);
                    return `<div class="bg-red-50 p-2 rounded flex justify-between"><span>⚠️ ${p.name}</span><span class="font-bold">${totalStock}/${p.threshold}</span></div>`;
                }).join('');
            }

            document.getElementById('totalItems').innerText = `Items: ${stock.reduce((sum, s) => sum + s.quantity, 0)}`;
        }

        // ============ LOCATIONS ============
        function renderLocations() {
            const search = document.getElementById('searchLocation')?.value.toLowerCase() || '';
            let filtered = locations.filter(l => l.name.toLowerCase().includes(search));

            const container = document.getElementById('locationsList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No locations found</div>';
                return;
            }

            container.innerHTML = filtered.map(l => {
                const locStock = stock.filter(s => s.locationId === l.id).reduce((sum, s) => sum + s.quantity, 0);
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${l.name}</h3>
                                <p class="text-xs text-gray-500">${l.city}, ${l.state}</p>
                                <p class="text-xs text-gray-500">${l.address}</p>
                            </div>
                            <button class="delete-location text-red-600 hover:text-red-800 font-bold" data-id="${l.id}">✕</button>
                        </div>
                        <div class="mb-3">
                            <p class="text-sm text-gray-500">Total Stock</p>
                            <p class="text-2xl font-bold">${locStock} units</p>
                        </div>
                        <div class="flex gap-2">
                            <button class="edit-location flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-id="${l.id}">✏️ Edit</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-location').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this location?')) {
                        locations = locations.filter(l => l.id !== btn.dataset.id);
                        stock = stock.filter(s => s.locationId !== btn.dataset.id);
                        saveData();
                    }
                });
            });

            document.querySelectorAll('.edit-location').forEach(btn => {
                btn.addEventListener('click', () => editLocation(btn.dataset.id));
            });
        }

        function editLocation(id) {
            const l = locations.find(x => x.id === id);
            if (!l) return;
            document.getElementById('locationName').value = l.name;
            document.getElementById('locationCity').value = l.city;
            document.getElementById('locationState').value = l.state;
            document.getElementById('locationAddress').value = l.address;
            document.getElementById('locationModalTitle').innerText = 'Edit Location';
            editingLocationId = id;
            document.getElementById('locationModal').classList.remove('hidden');
        }

        document.getElementById('addLocationBtn').addEventListener('click', () => {
            document.getElementById('locationName').value = '';
            document.getElementById('locationCity').value = '';
            document.getElementById('locationState').value = '';
            document.getElementById('locationAddress').value = '';
            document.getElementById('locationModalTitle').innerText = 'Add Location';
            editingLocationId = null;
            document.getElementById('locationModal').classList.remove('hidden');
        });

        document.getElementById('saveLocationBtn').addEventListener('click', () => {
            const name = document.getElementById('locationName').value.trim();
            const city = document.getElementById('locationCity').value.trim();
            const state = document.getElementById('locationState').value.trim();
            const address = document.getElementById('locationAddress').value.trim();

            if (!name || !city || !state) { alert('Name, City, and State required'); return; }

            if (editingLocationId) {
                const l = locations.find(x => x.id === editingLocationId);
                if (l) {
                    l.name = name;
                    l.city = city;
                    l.state = state;
                    l.address = address;
                }
            } else {
                locations.push({
                    id: Date.now().toString(),
                    name, city, state, address
                });
            }
            saveData();
            document.getElementById('locationModal').classList.add('hidden');
        });

        document.getElementById('closeLocationModal').addEventListener('click', () => {
            document.getElementById('locationModal').classList.add('hidden');
        });

        // ============ PRODUCTS ============
        function renderProducts() {
            const search = document.getElementById('searchProduct')?.value.toLowerCase() || '';
            let filtered = products.filter(p => p.name.toLowerCase().includes(search) || p.sku.toLowerCase().includes(search));

            const container = document.getElementById('productsList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No products found</div>';
                return;
            }

            container.innerHTML = filtered.map(p => {
                const totalStock = stock.filter(s => s.productId === p.id).reduce((sum, s) => sum + s.quantity, 0);
                const isLowStock = totalStock <= p.threshold;
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${p.name} ${isLowStock ? '<span class="text-red-600">●</span>' : ''}</h3>
                                <p class="text-xs text-gray-500">SKU: ${p.sku}</p>
                            </div>
                            <button class="delete-product text-red-600 hover:text-red-800 font-bold" data-id="${p.id}">✕</button>
                        </div>
                        <div class="grid grid-cols-3 gap-2 mb-3">
                            <div>
                                <p class="text-sm text-gray-500">Total Stock</p>
                                <p class="text-xl font-bold">${totalStock}</p>
                            </div>
                            <div>
                                <p class="text-sm text-gray-500">Price</p>
                                <p class="text-xl font-bold">$${p.price.toFixed(2)}</p>
                            </div>
                            <div>
                                <p class="text-sm text-gray-500">Threshold</p>
                                <p class="text-xl font-bold">${p.threshold}</p>
                            </div>
                        </div>
                        <div class="flex gap-2">
                            <button class="edit-product flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-id="${p.id}">✏️ Edit</button>
                            <button class="add-stock flex-1 bg-green-500 text-white px-3 py-1 rounded text-sm" data-id="${p.id}">➕ Add Stock</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-product').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this product?')) {
                        products = products.filter(p => p.id !== btn.dataset.id);
                        stock = stock.filter(s => s.productId !== btn.dataset.id);
                        saveData();
                    }
                });
            });

            document.querySelectorAll('.edit-product').forEach(btn => {
                btn.addEventListener('click', () => editProduct(btn.dataset.id));
            });

            document.querySelectorAll('.add-stock').forEach(btn => {
                btn.addEventListener('click', () => openAddStock(btn.dataset.id));
            });
        }

        function editProduct(id) {
            const p = products.find(x => x.id === id);
            if (!p) return;
            document.getElementById('productName').value = p.name;
            document.getElementById('productSku').value = p.sku;
            document.getElementById('productPrice').value = p.price;
            document.getElementById('productThreshold').value = p.threshold;
            document.getElementById('productModalTitle').innerText = 'Edit Product';
            editingProductId = id;
            document.getElementById('productModal').classList.remove('hidden');
        }

        function openAddStock(id) {
            currentStockProductId = id;
            document.getElementById('stockLocation').innerHTML = '<option>Select Location</option>' + 
                locations.map(l => `<option value="${l.id}">${l.name}</option>`).join('');
            document.getElementById('stockQuantity').value = '';
            document.getElementById('stockModal').classList.remove('hidden');
        }

        document.getElementById('addProductBtn').addEventListener('click', () => {
            document.getElementById('productName').value = '';
            document.getElementById('productSku').value = '';
            document.getElementById('productPrice').value = '';
            document.getElementById('productThreshold').value = '10';
            document.getElementById('productModalTitle').innerText = 'Add Product';
            editingProductId = null;
            document.getElementById('productModal').classList.remove('hidden');
        });

        document.getElementById('saveProductBtn').addEventListener('click', () => {
            const name = document.getElementById('productName').value.trim();
            const sku = document.getElementById('productSku').value.trim();
            const price = parseFloat(document.getElementById('productPrice').value) || 0;
            const threshold = parseInt(document.getElementById('productThreshold').value) || 10;

            if (!name || !sku) { alert('Name and SKU required'); return; }

            if (editingProductId) {
                const p = products.find(x => x.id === editingProductId);
                if (p) {
                    p.name = name;
                    p.sku = sku;
                    p.price = price;
                    p.threshold = threshold;
                }
            } else {
                products.push({
                    id: Date.now().toString(),
                    name, sku, price, threshold
                });
            }
            saveData();
            document.getElementById('productModal').classList.add('hidden');
        });

        document.getElementById('closeProductModal').addEventListener('click', () => {
            document.getElementById('productModal').classList.add('hidden');
        });

        document.getElementById('saveStockBtn').addEventListener('click', () => {
            const locationId = document.getElementById('stockLocation').value;
            const quantity = parseInt(document.getElementById('stockQuantity').value) || 0;

            if (!locationId || locationId === 'Select Location' || !quantity || quantity <= 0) {
                alert('Select location and enter valid quantity');
                return;
            }

            stock.push({
                id: Date.now().toString(),
                productId: currentStockProductId,
                locationId: locationId,
                quantity: quantity,
                date: new Date().toISOString()
            });

            saveData();
            document.getElementById('stockModal').classList.add('hidden');
        });

        document.getElementById('closeStockModal').addEventListener('click', () => {
            document.getElementById('stockModal').classList.add('hidden');
        });

        // ============ TRANSFERS ============
        function renderTransfers() {
            const search = document.getElementById('searchTransfer')?.value.toLowerCase() || '';
            let filtered = transfers.filter(t => {
                const prod = products.find(p => p.id === t.productId);
                const fromLoc = locations.find(l => l.id === t.fromLocationId);
                const toLoc = locations.find(l => l.id === t.toLocationId);
                return (prod?.name.toLowerCase().includes(search) || fromLoc?.name.toLowerCase().includes(search) || toLoc?.name.toLowerCase().includes(search));
            });

            const container = document.getElementById('transfersList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No transfers found</div>';
                return;
            }

            container.innerHTML = filtered.map(t => {
                const prod = products.find(p => p.id === t.productId);
                const fromLoc = locations.find(l => l.id === t.fromLocationId);
                const toLoc = locations.find(l => l.id === t.toLocationId);
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold">${prod?.name || 'Unknown'} × ${t.quantity}</h3>
                                <p class="text-sm text-gray-600">From: ${fromLoc?.name || 'Unknown'}</p>
                                <p class="text-sm text-gray-600">To: ${toLoc?.name || 'Unknown'}</p>
                            </div>
                            <button class="delete-transfer text-red-600 hover:text-red-800 font-bold" data-id="${t.id}">✕</button>
                        </div>
                        <div class="text-xs text-gray-400">${new Date(t.date).toLocaleString()}</div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-transfer').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this transfer?')) {
                        transfers = transfers.filter(t => t.id !== btn.dataset.id);
                        saveData();
                    }
                });
            });
        }

        document.getElementById('addTransferBtn').addEventListener('click', () => {
            document.getElementById('transferProduct').innerHTML = '<option>Select Product</option>' + 
                products.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
            document.getElementById('transferFrom').innerHTML = '<option>From Location</option>' + 
                locations.map(l => `<option value="${l.id}">${l.name}</option>`).join('');
            document.getElementById('transferTo').innerHTML = '<option>To Location</option>' + 
                locations.map(l => `<option value="${l.id}">${l.name}</option>`).join('');
            document.getElementById('transferQuantity').value = '';
            document.getElementById('transferModal').classList.remove('hidden');
        });

        document.getElementById('saveTransferBtn').addEventListener('click', () => {
            const productId = document.getElementById('transferProduct').value;
            const fromLocationId = document.getElementById('transferFrom').value;
            const toLocationId = document.getElementById('transferTo').value;
            const quantity = parseInt(document.getElementById('transferQuantity').value) || 0;

            if (!productId || !fromLocationId || !toLocationId || !quantity) {
                alert('All fields required');
                return;
            }

            if (fromLocationId === toLocationId) {
                alert('Cannot transfer to the same location');
                return;
            }

            const fromStock = stock.find(s => s.productId === productId && s.locationId === fromLocationId);
            if (!fromStock || fromStock.quantity < quantity) {
                alert('Not enough stock at source location');
                return;
            }

            fromStock.quantity -= quantity;

            const toStock = stock.find(s => s.productId === productId && s.locationId === toLocationId);
            if (toStock) {
                toStock.quantity += quantity;
            } else {
                stock.push({
                    id: Date.now().toString(),
                    productId,
                    locationId: toLocationId,
                    quantity,
                    date: new Date().toISOString()
                });
            }

            transfers.push({
                id: Date.now().toString(),
                productId,
                fromLocationId,
                toLocationId,
                quantity,
                date: new Date().toISOString()
            });

            saveData();
            document.getElementById('transferModal').classList.add('hidden');
        });

        document.getElementById('closeTransferModal').addEventListener('click', () => {
            document.getElementById('transferModal').classList.add('hidden');
        });

        // ============ DELIVERIES ============
        function renderDeliveries() {
            const search = document.getElementById('searchDelivery')?.value.toLowerCase() || '';
            let filtered = deliveries.filter(d => {
                const prod = products.find(p => p.id === d.productId);
                const loc = locations.find(l => l.id === d.locationId);
                return (prod?.name.toLowerCase().includes(search) || loc?.name.toLowerCase().includes(search) || d.customer.toLowerCase().includes(search));
            });

            const container = document.getElementById('deliveriesList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No deliveries found</div>';
                return;
            }

            container.innerHTML = filtered.map(d => {
                const prod = products.find(p => p.id === d.productId);
                const loc = locations.find(l => l.id === d.locationId);
                const statusColor = d.status === 'completed' ? 'bg-green-100' : d.status === 'failed' ? 'bg-red-100' : 'bg-yellow-100';
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold">${prod?.name || 'Unknown'}</h3>
                                <p class="text-sm text-gray-600">📍 ${loc?.name || 'Unknown'}</p>
                                <p class="text-sm text-gray-600">👤 ${d.customer}</p>
                                <p class="text-xs text-gray-500">${d.address}</p>
                            </div>
                            <span class="text-xs font-bold ${statusColor} px-2 py-1 rounded">${d.status.toUpperCase()}</span>
                        </div>
                        <div class="grid grid-cols-2 gap-2 mb-2">
                            <div>
                                <p class="text-xs text-gray-500">In Stock</p>
                                <p class="font-bold">${d.quantityInStock}</p>
                            </div>
                            <div>
                                <p class="text-xs text-gray-500">Delivered</p>
                                <p class="font-bold text-blue-600">${d.quantityDelivered}</p>
                            </div>
                        </div>
                        <div class="text-xs text-gray-400 mb-2">${new Date(d.date).toLocaleString()}</div>
                        <div class="flex gap-2">
                            <select class="update-delivery-status flex-1 border p-1 rounded text-sm" data-id="${d.id}">
                                <option value="pending" ${d.status === 'pending' ? 'selected' : ''}>Pending</option>
                                <option value="completed" ${d.status === 'completed' ? 'selected' : ''}>Completed</option>
                                <option value="failed" ${d.status === 'failed' ? 'selected' : ''}>Failed</option>
                            </select>
                            <button class="edit-delivery bg-blue-500 text-white px-3 py-1 rounded text-sm" data-id="${d.id}">✏️ Edit</button>
                            <button class="delete-delivery bg-red-500 text-white px-3 py-1 rounded text-sm" data-id="${d.id}">✕</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.update-delivery-status').forEach(select => {
                select.addEventListener('change', () => {
                    const d = deliveries.find(x => x.id === select.dataset.id);
                    if (d) { d.status = select.value; saveData(); }
                });
            });

            document.querySelectorAll('.edit-delivery').forEach(btn => {
                btn.addEventListener('click', () => editDelivery(btn.dataset.id));
            });

            document.querySelectorAll('.delete-delivery').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete delivery?')) {
                        deliveries = deliveries.filter(d => d.id !== btn.dataset.id);
                        saveData();
                    }
                });
            });
        }

        function editDelivery(id) {
            const d = deliveries.find(x => x.id === id);
            if (!d) return;
            document.getElementById('deliveryLocation').innerHTML = '<option>Select Location</option>' + 
                locations.map(l => `<option value="${l.id}" ${l.id === d.locationId ? 'selected' : ''}>${l.name}</option>`).join('');
            document.getElementById('deliveryProduct').innerHTML = '<option>Select Product</option>' + 
                products.map(p => `<option value="${p.id}" ${p.id === d.productId ? 'selected' : ''}>${p.name}</option>`).join('');
            document.getElementById('deliveryQuantityInStock').value = d.quantityInStock;
            document.getElementById('deliveryQuantityDelivered').value = d.quantityDelivered;
            document.getElementById('deliveryCustomer').value = d.customer;
            document.getElementById('deliveryAddress').value = d.address;
            document.getElementById('deliveryStatus').value = d.status;
            document.getElementById('deliveryModalTitle').innerText = 'Edit Delivery';
            editingDeliveryId = id;
            document.getElementById('deliveryModal').classList.remove('hidden');
        }

        document.getElementById('addDeliveryBtn').addEventListener('click', () => {
            document.getElementById('deliveryLocation').innerHTML = '<option>Select Location</option>' + 
                locations.map(l => `<option value="${l.id}">${l.name}</option>`).join('');
            document.getElementById('deliveryProduct').innerHTML = '<option>Select Product</option>' + 
                products.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
            document.getElementById('deliveryQuantityInStock').value = '';
            document.getElementById('deliveryQuantityDelivered').value = '';
            document.getElementById('deliveryCustomer').value = '';
            document.getElementById('deliveryAddress').value = '';
            document.getElementById('deliveryStatus').value = 'pending';
            document.getElementById('deliveryModalTitle').innerText = 'Record Delivery';
            editingDeliveryId = null;
            document.getElementById('deliveryModal').classList.remove('hidden');
        });

        document.getElementById('deliveryProduct').addEventListener('change', function() {
            const productId = this.value;
            const locationId = document.getElementById('deliveryLocation').value;
            if (productId && locationId) {
                const s = stock.find(st => st.productId === productId && st.locationId === locationId);
                document.getElementById('deliveryQuantityInStock').value = s ? s.quantity : 0;
            }
        });

        document.getElementById('saveDeliveryBtn').addEventListener('click', () => {
            const locationId = document.getElementById('deliveryLocation').value;
            const productId = document.getElementById('deliveryProduct').value;
            const quantityInStock = parseInt(document.getElementById('deliveryQuantityInStock').value) || 0;
            const quantityDelivered = parseInt(document.getElementById('deliveryQuantityDelivered').value) || 0;
            const customer = document.getElementById('deliveryCustomer').value.trim();
            const address = document.getElementById('deliveryAddress').value.trim();
            const status = document.getElementById('deliveryStatus').value;

            if (!locationId || !productId || !quantityDelivered || !customer) {
                alert('All fields required');
                return;
            }

            if (quantityDelivered > quantityInStock) {
                alert('Delivery quantity cannot exceed stock');
                return;
            }

            if (editingDeliveryId) {
                const d = deliveries.find(x => x.id === editingDeliveryId);
                if (d) {
                    d.locationId = locationId;
                    d.productId = productId;
                    d.quantityInStock = quantityInStock;
                    d.quantityDelivered = quantityDelivered;
                    d.customer = customer;
                    d.address = address;
                    d.status = status;
                }
            } else {
                deliveries.push({
                    id: Date.now().toString(),
                    locationId,
                    productId,
                    quantityInStock,
                    quantityDelivered,
                    customer,
                    address,
                    status,
                    date: new Date().toISOString()
                });
            }

            saveData();
            document.getElementById('deliveryModal').classList.add('hidden');
        });

        document.getElementById('closeDeliveryModal').addEventListener('click', () => {
            document.getElementById('deliveryModal').classList.add('hidden');
        });

        // ============ TAB SWITCHING ============
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                const tab = btn.dataset.tab;
                document.querySelectorAll('.tab-content').forEach(t => t.classList.add('hidden'));
                document.getElementById(tab + 'Tab').classList.remove('hidden');
                document.querySelectorAll('.tab-btn').forEach(b => {
                    b.classList.remove('bg-blue-600', 'text-white');
                    b.classList.add('bg-gray-200', 'text-gray-700');
                });
                btn.classList.remove('bg-gray-200', 'text-gray-700');
                btn.classList.add('bg-blue-600', 'text-white');
            });
        });

        // ============ SEARCH ============
        document.getElementById('searchLocation').addEventListener('input', renderLocations);
        document.getElementById('searchProduct').addEventListener('input', renderProducts);
        document.getElementById('searchTransfer').addEventListener('input', renderTransfers);
        document.getElementById('searchDelivery').addEventListener('input', renderDeliveries);

        // ============ EXPORT ============
        document.getElementById('exportProductsBtn').addEventListener('click', () => {
            const data = products.map(p => {
                const totalStock = stock.filter(s => s.productId === p.id).reduce((sum, s) => sum + s.quantity, 0);
                return {
                    Name: p.name,
                    SKU: p.sku,
                    'Total Stock': totalStock,
                    Price: p.price,
                    Threshold: p.threshold
                };
            });
            const ws = XLSX.utils.json_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Products');
            XLSX.writeFile(wb, 'stockflow-products.xlsx');
        });

        // ============ CLEAR ALL ============
        document.getElementById('clearAllBtn').addEventListener('click', () => {
            if (confirm('⚠️ This will delete ALL data. Are you sure?')) {
                locations = [];
                products = [];
                stock = [];
                transfers = [];
                deliveries = [];
                saveData();
            }
        });

        // ============ INITIALIZE ============
        updateDashboard();
        renderLocations();
        renderProducts();
        renderTransfers();
        renderDeliveries();
    </script>
</body>
</html>
