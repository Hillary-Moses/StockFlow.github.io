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
        .location-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 600;
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
            <div>
                <h1 class="text-2xl font-bold text-blue-800">📦 StockFlow</h1>
                <p class="text-xs text-gray-500">Multi-Location Inventory Management</p>
            </div>
            <div class="flex gap-2 items-center">
                <span id="totalItems" class="text-sm bg-blue-100 px-3 py-1 rounded-full">Items: 0</span>
                <button id="clearAllBtn" class="bg-red-500 text-white px-3 py-1 rounded text-sm">Clear All</button>
            </div>
        </div>

        <!-- Tab Navigation -->
        <div class="grid grid-cols-5 gap-2 mb-6 bg-white p-2 rounded-xl shadow overflow-x-auto">
            <button data-tab="dashboard" class="tab-btn py-3 rounded-xl font-semibold bg-blue-600 text-white whitespace-nowrap">📊 Dashboard</button>
            <button data-tab="locations" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700 whitespace-nowrap">📍 Locations</button>
            <button data-tab="products" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700 whitespace-nowrap">📦 Products</button>
            <button data-tab="deliveries" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700 whitespace-nowrap">🚚 Deliveries</button>
            <button data-tab="transfers" class="tab-btn py-3 rounded-xl font-semibold bg-gray-200 text-gray-700 whitespace-nowrap">↔️ Transfers</button>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboardTab" class="tab-content">
            <div class="grid grid-cols-2 gap-3 mb-5">
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">📦 Total Products</p>
                    <p id="totalProducts" class="text-3xl font-bold">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">📍 Locations</p>
                    <p id="totalLocations" class="text-3xl font-bold">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">⚠️ Low Stock Items</p>
                    <p id="lowStockAlert" class="text-3xl font-bold text-red-600">0</p>
                </div>
                <div class="bg-white rounded-xl p-4 shadow text-center">
                    <p class="text-gray-500 text-sm">💼 Total Stock Value</p>
                    <p id="totalValue" class="text-3xl font-bold text-green-600">$0</p>
                </div>
            </div>

            <!-- Stock by Location -->
            <div class="bg-white rounded-xl p-4 shadow mb-4">
                <h3 class="font-bold text-lg mb-3">📊 Stock Distribution by Location</h3>
                <div id="stockByLocation" class="space-y-2"></div>
            </div>

            <!-- Low Stock Alerts -->
            <div class="bg-white rounded-xl p-4 shadow">
                <h3 class="font-bold text-lg mb-3">⚠️ Low Stock Alerts</h3>
                <div id="lowStockList" class="space-y-2"></div>
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

        <!-- Deliveries Tab -->
        <div id="deliveriesTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="recordDeliveryBtn" class="bg-amber-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">📤 Record Delivery</button>
                <input type="text" id="searchDelivery" placeholder="🔍 Search deliveries..." class="flex-1 border p-2 rounded-xl">
            </div>
            <div id="deliveriesList" class="space-y-3"></div>
        </div>

        <!-- Transfers Tab -->
        <div id="transfersTab" class="tab-content hidden">
            <div class="flex gap-2 mb-4">
                <button id="newTransferBtn" class="bg-purple-600 text-white px-4 py-2 rounded-xl flex-1 btn-large">↔️ New Transfer</button>
                <input type="text" id="searchTransfer" placeholder="🔍 Search transfers..." class="flex-1 border p-2 rounded-xl">
            </div>
            <div id="transfersList" class="space-y-3"></div>
        </div>

    </div>

    <!-- Location Modal -->
    <div id="locationModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4">Add Location</h3>
            <input id="locationName" placeholder="Location Name (e.g., Ibadan HQ)" class="w-full border p-2 rounded mb-2">
            <input id="locationCity" placeholder="City" class="w-full border p-2 rounded mb-2">
            <input id="locationState" placeholder="State" class="w-full border p-2 rounded mb-2">
            <textarea id="locationAddress" placeholder="Full Address" class="w-full border p-2 rounded mb-4" rows="3"></textarea>
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
            <input id="productSku" placeholder="SKU/Code" class="w-full border p-2 rounded mb-2">
            <input id="productPrice" type="number" placeholder="Price per Unit" class="w-full border p-2 rounded mb-2">
            <input id="productThreshold" type="number" placeholder="Low Stock Threshold" class="w-full border p-2 rounded mb-4" value="10">
            <div class="flex gap-2">
                <button id="saveProductBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button>
                <button id="closeProductModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Stock Add Modal -->
    <div id="stockModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4">Add Stock</h3>
            <p id="stockModalProduct" class="text-sm text-gray-600 mb-2"></p>
            <select id="stockLocation" class="w-full border p-2 rounded mb-2">
                <option>Select Location</option>
            </select>
            <input id="stockQuantity" type="number" placeholder="Quantity to Add" class="w-full border p-2 rounded mb-4">
            <div class="flex gap-2">
                <button id="saveStockBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Save</button>
                <button id="closeStockModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Delivery Recording Modal -->
    <div id="deliveryModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5 max-h-[90vh] overflow-y-auto">
            <h3 class="text-xl font-bold mb-4" id="deliveryModalTitle">Record Delivery</h3>

            <label class="block text-sm font-semibold mb-2">Select Location</label>
            <select id="deliveryLocation" class="w-full border p-2 rounded mb-4">
                <option>Select Location</option>
            </select>

            <label class="block text-sm font-semibold mb-2">Select Product</label>
            <select id="deliveryProduct" class="w-full border p-2 rounded mb-4">
                <option>Select Product</option>
            </select>

            <label class="block text-sm font-semibold mb-2">Current Stock</label>
            <input id="currentStockDisplay" type="text" class="w-full border p-2 rounded mb-4 bg-gray-100" readonly>

            <label class="block text-sm font-semibold mb-2">Quantity Delivered</label>
            <input id="deliveryQuantity" type="number" placeholder="Enter quantity delivered" class="w-full border p-2 rounded mb-2" min="0">

            <label class="block text-sm font-semibold mb-2">Customer Name</label>
            <input id="customerName" placeholder="Customer name" class="w-full border p-2 rounded mb-2">

            <label class="block text-sm font-semibold mb-2">Customer Address</label>
            <textarea id="customerAddress" placeholder="Delivery address" class="w-full border p-2 rounded mb-2" rows="2"></textarea>

            <label class="block text-sm font-semibold mb-2">Status</label>
            <select id="deliveryStatus" class="w-full border p-2 rounded mb-4">
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
                <option value="failed">Failed</option>
            </select>

            <div class="flex gap-2">
                <button id="saveDeliveryBtn" class="bg-blue-600 text-white px-4 py-2 rounded flex-1">Record Delivery</button>
                <button id="closeDeliveryModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Transfer Modal -->
    <div id="transferModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl max-w-md w-full p-5">
            <h3 class="text-xl font-bold mb-4" id="transferModalTitle">Transfer Stock</h3>

            <label class="block text-sm font-semibold mb-2">Select Product</label>
            <select id="transferProduct" class="w-full border p-2 rounded mb-4">
                <option>Select Product</option>
            </select>

            <label class="block text-sm font-semibold mb-2">From Location</label>
            <select id="transferFrom" class="w-full border p-2 rounded mb-4">
                <option>Select Location</option>
            </select>

            <label class="block text-sm font-semibold mb-2">Current Stock at Source</label>
            <input id="transferFromStock" type="text" class="w-full border p-2 rounded mb-4 bg-gray-100" readonly>

            <label class="block text-sm font-semibold mb-2">To Location</label>
            <select id="transferTo" class="w-full border p-2 rounded mb-4">
                <option>Select Location</option>
            </select>

            <label class="block text-sm font-semibold mb-2">Quantity to Transfer</label>
            <input id="transferQuantity" type="number" placeholder="Enter quantity" class="w-full border p-2 rounded mb-4" min="0">

            <div class="flex gap-2">
                <button id="saveTransferBtn" class="bg-purple-600 text-white px-4 py-2 rounded flex-1">Transfer</button>
                <button id="closeTransferModal" class="bg-gray-400 px-4 py-2 rounded flex-1">Cancel</button>
            </div>
        </div>
    </div>

    <script>
        // ============ LOCAL STORAGE DATA ============
        let locations = JSON.parse(localStorage.getItem('stockflow_locations')) || [];
        let products = JSON.parse(localStorage.getItem('stockflow_products')) || [];
        let inventory = JSON.parse(localStorage.getItem('stockflow_inventory')) || {};
        let deliveries = JSON.parse(localStorage.getItem('stockflow_deliveries')) || [];
        let transfers = JSON.parse(localStorage.getItem('stockflow_transfers')) || [];

        // ============ SAVE TO LOCAL STORAGE ============
        function saveData() {
            localStorage.setItem('stockflow_locations', JSON.stringify(locations));
            localStorage.setItem('stockflow_products', JSON.stringify(products));
            localStorage.setItem('stockflow_inventory', JSON.stringify(inventory));
            localStorage.setItem('stockflow_deliveries', JSON.stringify(deliveries));
            localStorage.setItem('stockflow_transfers', JSON.stringify(transfers));
            updateDashboard();
            renderLocations();
            renderProducts();
            renderDeliveries();
            renderTransfers();
        }

        // ============ TAB NAVIGATION ============
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                const tabName = btn.dataset.tab;
                document.querySelectorAll('.tab-content').forEach(t => t.classList.add('hidden'));
                document.getElementById(tabName + 'Tab').classList.remove('hidden');
                document.querySelectorAll('.tab-btn').forEach(b => {
                    b.classList.remove('bg-blue-600', 'text-white');
                    b.classList.add('bg-gray-200', 'text-gray-700');
                });
                btn.classList.remove('bg-gray-200', 'text-gray-700');
                btn.classList.add('bg-blue-600', 'text-white');
            });
        });

        // ============ DASHBOARD ============
        function updateDashboard() {
            document.getElementById('totalProducts').innerText = products.length;
            document.getElementById('totalLocations').innerText = locations.length;

            let lowStockItems = [];
            products.forEach(p => {
                let totalQty = 0;
                if (inventory[p.id]) {
                    Object.values(inventory[p.id]).forEach(qty => totalQty += qty);
                }
                if (totalQty <= p.threshold) {
                    lowStockItems.push(p);
                }
            });
            document.getElementById('lowStockAlert').innerText = lowStockItems.length;

            let totalVal = 0;
            products.forEach(p => {
                let totalQty = 0;
                if (inventory[p.id]) {
                    Object.values(inventory[p.id]).forEach(qty => totalQty += qty);
                }
                totalVal += totalQty * p.price;
            });
            document.getElementById('totalValue').innerText = '$' + totalVal.toFixed(2);

            const stockByLocContainer = document.getElementById('stockByLocation');
            if (locations.length === 0) {
                stockByLocContainer.innerHTML = '<div class="text-gray-500">No locations added yet</div>';
            } else {
                stockByLocContainer.innerHTML = locations.map(loc => {
                    let locTotal = 0;
                    Object.keys(inventory).forEach(prodId => {
                        if (inventory[prodId][loc.id]) {
                            locTotal += inventory[prodId][loc.id];
                        }
                    });
                    return `<div class="bg-gray-50 p-3 rounded flex justify-between">
                        <span class="font-semibold">${loc.name} (${loc.city})</span>
                        <span class="bg-blue-200 text-blue-800 px-3 py-1 rounded-full">${locTotal} units</span>
                    </div>`;
                }).join('');
            }

            const lowStockContainer = document.getElementById('lowStockList');
            if (lowStockItems.length === 0) {
                lowStockContainer.innerHTML = '<div class="text-gray-500">✅ All stock levels healthy</div>';
            } else {
                lowStockContainer.innerHTML = lowStockItems.map(p => {
                    let details = '';
                    if (inventory[p.id]) {
                        details = locations.map(loc => {
                            const qty = inventory[p.id][loc.id] || 0;
                            return `<span class="text-xs bg-gray-200 px-2 py-1 rounded">${loc.city}: ${qty}</span>`;
                        }).join(' ');
                    }
                    return `<div class="bg-red-50 p-3 rounded"><div class="font-bold mb-2">⚠️ ${p.name}</div><div class="flex gap-2 flex-wrap">${details}</div></div>`;
                }).join('');
            }

            document.getElementById('totalItems').innerText = `Items: ${Object.keys(inventory).reduce((sum, prodId) => {
                let qty = 0;
                if (inventory[prodId]) Object.values(inventory[prodId]).forEach(q => qty += q);
                return sum + qty;
            }, 0)}`;
        }

        // ============ LOCATIONS ============
        function renderLocations() {
            const search = document.getElementById('searchLocation')?.value.toLowerCase() || '';
            let filtered = locations.filter(l =>
                l.name.toLowerCase().includes(search) || l.city.toLowerCase().includes(search)
            );

            const container = document.getElementById('locationsList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No locations found</div>';
                return;
            }

            container.innerHTML = filtered.map(loc => {
                let locTotal = 0;
                Object.keys(inventory).forEach(prodId => {
                    if (inventory[prodId][loc.id]) {
                        locTotal += inventory[prodId][loc.id];
                    }
                });
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${loc.name}</h3>
                                <p class="text-sm text-gray-600">📍 ${loc.city}, ${loc.state}</p>
                                <p class="text-xs text-gray-500">${loc.address}</p>
                            </div>
                            <button class="delete-location text-red-600 font-bold" data-id="${loc.id}">✕</button>
                        </div>
                        <div class="bg-blue-100 text-blue-800 px-3 py-2 rounded mb-2 text-center font-bold">
                            Total Stock: ${locTotal} units
                        </div>
                        <div class="flex gap-2">
                            <button class="edit-location flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-id="${loc.id}">✏️ Edit</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-location').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this location?')) {
                        locations = locations.filter(l => l.id !== btn.dataset.id);
                        saveData();
                    }
                });
            });
        }

        document.getElementById('addLocationBtn').addEventListener('click', () => {
            document.getElementById('locationName').value = '';
            document.getElementById('locationCity').value = '';
            document.getElementById('locationState').value = '';
            document.getElementById('locationAddress').value = '';
            document.getElementById('locationModal').classList.remove('hidden');
        });

        document.getElementById('saveLocationBtn').addEventListener('click', () => {
            const name = document.getElementById('locationName').value.trim();
            const city = document.getElementById('locationCity').value.trim();
            const state = document.getElementById('locationState').value.trim();
            const address = document.getElementById('locationAddress').value.trim();

            if (!name || !city || !state) { alert('Name, City, and State required'); return; }

            locations.push({
                id: Date.now().toString(),
                name, city, state, address
            });
            saveData();
            document.getElementById('locationModal').classList.add('hidden');
        });

        document.getElementById('closeLocationModal').addEventListener('click', () => {
            document.getElementById('locationModal').classList.add('hidden');
        });

        document.getElementById('searchLocation').addEventListener('input', renderLocations);

        // ============ PRODUCTS ============
        function renderProducts() {
            const search = document.getElementById('searchProduct')?.value.toLowerCase() || '';
            let filtered = products.filter(p =>
                p.name.toLowerCase().includes(search) || p.sku.toLowerCase().includes(search)
            );

            const container = document.getElementById('productsList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No products found</div>';
                return;
            }

            container.innerHTML = filtered.map(p => {
                let totalQty = 0;
                let locationBreakdown = '';
                if (inventory[p.id]) {
                    locations.forEach(loc => {
                        const qty = inventory[p.id][loc.id] || 0;
                        totalQty += qty;
                        const statusClass = qty <= p.threshold ? 'bg-red-100 text-red-800' : 'bg-green-100 text-green-800';
                        locationBreakdown += `<span class="${statusClass} text-xs px-2 py-1 rounded">${loc.city}: ${qty}</span> `;
                    });
                } else {
                    locations.forEach(loc => {
                        locationBreakdown += `<span class="bg-gray-100 text-gray-800 text-xs px-2 py-1 rounded">${loc.city}: 0</span> `;
                    });
                }

                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${p.name}</h3>
                                <p class="text-xs text-gray-500">SKU: ${p.sku}</p>
                            </div>
                            <button class="delete-product text-red-600 font-bold" data-id="${p.id}">✕</button>
                        </div>
                        <div class="grid grid-cols-2 gap-2 mb-3">
                            <div class="bg-blue-50 p-2 rounded">
                                <p class="text-xs text-gray-600">Total Stock</p>
                                <p class="text-lg font-bold">${totalQty}</p>
                            </div>
                            <div class="bg-gray-50 p-2 rounded">
                                <p class="text-xs text-gray-600">Unit Price</p>
                                <p class="text-lg font-bold">$${p.price.toFixed(2)}</p>
                            </div>
                        </div>
                        <div class="mb-3 flex gap-1 flex-wrap">
                            ${locationBreakdown}
                        </div>
                        <div class="flex gap-2">
                            <button class="add-stock flex-1 bg-green-500 text-white px-3 py-1 rounded text-sm" data-id="${p.id}">➕ Add Stock</button>
                            <button class="edit-product flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-id="${p.id}">✏️ Edit</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-product').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this product?')) {
                        const prodId = btn.dataset.id;
                        products = products.filter(p => p.id !== prodId);
                        delete inventory[prodId];
                        saveData();
                    }
                });
            });

            document.querySelectorAll('.add-stock').forEach(btn => {
                btn.addEventListener('click', () => {
                    window._stockProductId = btn.dataset.id;
                    const prod = products.find(p => p.id === window._stockProductId);
                    document.getElementById('stockModalProduct').innerText = `Add stock for: ${prod.name}`;

                    const select = document.getElementById('stockLocation');
                    select.innerHTML = '<option>Select Location</option>' +
                        locations.map(l => `<option value="${l.id}">${l.name} (${l.city})</option>`).join('');

                    document.getElementById('stockQuantity').value = '';
                    document.getElementById('stockModal').classList.remove('hidden');
                });
            });
        }

        document.getElementById('addProductBtn').addEventListener('click', () => {
            document.getElementById('productModalTitle').innerText = 'Add Product';
            document.getElementById('productName').value = '';
            document.getElementById('productSku').value = '';
            document.getElementById('productPrice').value = '';
            document.getElementById('productThreshold').value = '10';
            window._editProductId = null;
            document.getElementById('productModal').classList.remove('hidden');
        });

        document.getElementById('saveProductBtn').addEventListener('click', () => {
            const name = document.getElementById('productName').value.trim();
            const sku = document.getElementById('productSku').value.trim();
            const price = parseFloat(document.getElementById('productPrice').value) || 0;
            const threshold = parseInt(document.getElementById('productThreshold').value) || 10;

            if (!name || !sku) { alert('Name and SKU required'); return; }

            const productId = Date.now().toString();
            products.push({ id: productId, name, sku, price, threshold });
            inventory[productId] = {};
            locations.forEach(loc => {
                inventory[productId][loc.id] = 0;
            });
            saveData();
            document.getElementById('productModal').classList.add('hidden');
        });

        document.getElementById('closeProductModal').addEventListener('click', () => {
            document.getElementById('productModal').classList.add('hidden');
        });

        document.getElementById('searchProduct').addEventListener('input', renderProducts);

        // ============ STOCK UPDATE ============
        document.getElementById('saveStockBtn').addEventListener('click', () => {
            const prodId = window._stockProductId;
            const locId = document.getElementById('stockLocation').value;
            const qty = parseInt(document.getElementById('stockQuantity').value) || 0;

            if (!locId || locId === 'Select Location' || !qty || qty <= 0) {
                alert('Select location and enter valid quantity');
                return;
            }

            if (!inventory[prodId]) inventory[prodId] = {};
            inventory[prodId][locId] = (inventory[prodId][locId] || 0) + qty;

            saveData();
            document.getElementById('stockModal').classList.add('hidden');
        });

        document.getElementById('closeStockModal').addEventListener('click', () => {
            document.getElementById('stockModal').classList.add('hidden');
        });

        // ============ DELIVERIES ============
        function renderDeliveries() {
            const search = document.getElementById('searchDelivery')?.value.toLowerCase() || '';
            let filtered = deliveries.filter(d =>
                d.customerName.toLowerCase().includes(search) ||
                d.productName.toLowerCase().includes(search)
            );

            const container = document.getElementById('deliveriesList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No deliveries recorded</div>';
                return;
            }

            container.innerHTML = filtered.map((d, idx) => {
                const statusColors = {
                    'pending': 'bg-yellow-100 text-yellow-800',
                    'completed': 'bg-green-100 text-green-800',
                    'failed': 'bg-red-100 text-red-800'
                };
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${d.productName}</h3>
                                <p class="text-sm text-gray-600">👤 ${d.customerName}</p>
                                <p class="text-xs text-gray-500">📍 ${d.locationName}</p>
                            </div>
                            <button class="delete-delivery text-red-600 font-bold text-lg" data-index="${idx}">✕</button>
                        </div>
                        <div class="grid grid-cols-2 gap-2 mb-3">
                            <div class="bg-blue-50 p-2 rounded">
                                <p class="text-xs text-gray-600">Qty Delivered</p>
                                <p class="text-lg font-bold">${d.quantityDelivered}</p>
                            </div>
                            <div class="bg-gray-50 p-2 rounded">
                                <p class="text-xs text-gray-600">Status</p>
                                <p class="text-sm font-bold ${statusColors[d.status] || 'bg-gray-200'} px-2 py-1 rounded text-center">${d.status}</p>
                            </div>
                        </div>
                        <p class="text-xs text-gray-500 mb-3">📅 ${new Date(d.date).toLocaleString()}</p>
                        <div class="flex gap-2">
                            <button class="edit-delivery flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-index="${idx}">✏️ Edit</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-delivery').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this delivery?')) {
                        const idx = parseInt(btn.dataset.index);
                        const d = deliveries[idx];
                        // Restore stock
                        if (inventory[d.productId] && inventory[d.productId][d.locationId]) {
                            inventory[d.productId][d.locationId] += d.quantityDelivered;
                        }
                        deliveries.splice(idx, 1);
                        saveData();
                    }
                });
            });

            document.querySelectorAll('.edit-delivery').forEach(btn => {
                btn.addEventListener('click', () => {
                    const idx = parseInt(btn.dataset.index);
                    const d = deliveries[idx];
                    window._editDeliveryIndex = idx;
                    document.getElementById('deliveryModalTitle').innerText = 'Edit Delivery';
                    document.getElementById('deliveryLocation').value = d.locationId;
                    document.getElementById('deliveryProduct').value = d.productId;
                    document.getElementById('currentStockDisplay').value = d.quantityDelivered;
                    document.getElementById('deliveryQuantity').value = d.quantityDelivered;
                    document.getElementById('customerName').value = d.customerName;
                    document.getElementById('customerAddress').value = d.customerAddress;
                    document.getElementById('deliveryStatus').value = d.status;
                    document.getElementById('deliveryModal').classList.remove('hidden');
                });
            });
        }

        document.getElementById('recordDeliveryBtn').addEventListener('click', () => {
            window._editDeliveryIndex = null;
            document.getElementById('deliveryModalTitle').innerText = 'Record Delivery';
            document.getElementById('deliveryLocation').value = 'Select Location';
            document.getElementById('deliveryProduct').value = 'Select Product';
            document.getElementById('currentStockDisplay').value = '';
            document.getElementById('deliveryQuantity').value = '';
            document.getElementById('customerName').value = '';
            document.getElementById('customerAddress').value = '';
            document.getElementById('deliveryStatus').value = 'pending';

            const locSelect = document.getElementById('deliveryLocation');
            locSelect.innerHTML = '<option>Select Location</option>' +
                locations.map(l => `<option value="${l.id}">${l.name} (${l.city})</option>`).join('');

            document.getElementById('deliveryModal').classList.remove('hidden');
        });

        document.getElementById('deliveryProduct').addEventListener('change', () => {
            const prodId = document.getElementById('deliveryProduct').value;
            const locId = document.getElementById('deliveryLocation').value;

            if (prodId && prodId !== 'Select Product' && locId && locId !== 'Select Location') {
                const stock = inventory[prodId]?.[locId] || 0;
                document.getElementById('currentStockDisplay').value = stock;
            }
        });

        document.getElementById('deliveryLocation').addEventListener('change', () => {
            const prodId = document.getElementById('deliveryProduct').value;
            const locId = document.getElementById('deliveryLocation').value;

            if (prodId && prodId !== 'Select Product' && locId && locId !== 'Select Location') {
                const stock = inventory[prodId]?.[locId] || 0;
                document.getElementById('currentStockDisplay').value = stock;
            }
        });

        document.getElementById('saveDeliveryBtn').addEventListener('click', () => {
            const locId = document.getElementById('deliveryLocation').value;
            const prodId = document.getElementById('deliveryProduct').value;
            const qty = parseInt(document.getElementById('deliveryQuantity').value) || 0;
            const custName = document.getElementById('customerName').value.trim();
            const custAddr = document.getElementById('customerAddress').value.trim();
            const status = document.getElementById('deliveryStatus').value;

            if (!locId || locId === 'Select Location' || !prodId || prodId === 'Select Product' || !qty || qty <= 0) {
                alert('Complete all fields');
                return;
            }

            const currentStock = inventory[prodId]?.[locId] || 0;

            // Editing existing delivery
            if (window._editDeliveryIndex !== null && window._editDeliveryIndex !== undefined) {
                const oldDelivery = deliveries[window._editDeliveryIndex];
                const oldQty = oldDelivery.quantityDelivered;

                // Restore old qty
                inventory[oldDelivery.productId][oldDelivery.locationId] += oldQty;

                // Check if enough stock for new qty
                const availableStock = inventory[prodId][locId];
                if (qty > availableStock) {
                    alert(`Not enough stock! Available: ${availableStock}`);
                    return;
                }

                // Deduct new qty
                inventory[prodId][locId] -= qty;

                deliveries[window._editDeliveryIndex] = {
                    id: oldDelivery.id,
                    productId: prodId,
                    locationId: locId,
                    productName: products.find(p => p.id === prodId).name,
                    locationName: locations.find(l => l.id === locId).name,
                    quantityDelivered: qty,
                    customerName: custName,
                    customerAddress: custAddr,
                    status,
                    date: oldDelivery.date
                };
            } else {
                // New delivery
                if (qty > currentStock) {
                    alert(`Not enough stock! Available: ${currentStock}`);
                    return;
                }

                inventory[prodId][locId] -= qty;

                deliveries.push({
                    id: Date.now().toString(),
                    productId: prodId,
                    locationId: locId,
                    productName: products.find(p => p.id === prodId).name,
                    locationName: locations.find(l => l.id === locId).name,
                    quantityDelivered: qty,
                    customerName: custName,
                    customerAddress: custAddr,
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

        document.getElementById('searchDelivery').addEventListener('input', renderDeliveries);

        // ============ TRANSFERS ============
        function renderTransfers() {
            const search = document.getElementById('searchTransfer')?.value.toLowerCase() || '';
            let filtered = transfers.filter(t =>
                t.productName.toLowerCase().includes(search) ||
                t.fromLocation.toLowerCase().includes(search) ||
                t.toLocation.toLowerCase().includes(search)
            );

            const container = document.getElementById('transfersList');
            if (filtered.length === 0) {
                container.innerHTML = '<div class="text-center p-4 bg-white rounded">No transfers recorded</div>';
                return;
            }

            container.innerHTML = filtered.map((t, idx) => {
                return `
                    <div class="bg-white p-4 rounded-xl shadow">
                        <div class="flex justify-between items-start mb-2">
                            <div>
                                <h3 class="font-bold text-lg">${t.productName}</h3>
                                <p class="text-sm text-gray-600">📦 ${t.quantity} units</p>
                            </div>
                            <button class="delete-transfer text-red-600 font-bold text-lg" data-index="${idx}">✕</button>
                        </div>
                        <div class="bg-blue-50 p-2 rounded mb-2">
                            <p class="text-xs text-gray-600">From → To</p>
                            <p class="text-sm font-bold">${t.fromLocation} ➡️ ${t.toLocation}</p>
                        </div>
                        <p class="text-xs text-gray-500">📅 ${new Date(t.date).toLocaleString()}</p>
                        <div class="flex gap-2 mt-3">
                            <button class="edit-transfer flex-1 bg-blue-500 text-white px-3 py-1 rounded text-sm" data-index="${idx}">✏️ Edit</button>
                        </div>
                    </div>
                `;
            }).join('');

            document.querySelectorAll('.delete-transfer').forEach(btn => {
                btn.addEventListener('click', () => {
                    if (confirm('Delete this transfer?')) {
                        const idx = parseInt(btn.dataset.index);
                        const t = transfers[idx];
                        // Reverse transfer
                        inventory[t.productId][t.fromLocationId] += t.quantity;
                        inventory[t.productId][t.toLocationId] -= t.quantity;
                        transfers.splice(idx, 1);
                        saveData();
                    }
                });
            });

            document.querySelectorAll('.edit-transfer').forEach(btn => {
                btn.addEventListener('click', () => {
                    const idx = parseInt(btn.dataset.index);
                    const t = transfers[idx];
                    window._editTransferIndex = idx;
                    document.getElementById('transferModalTitle').innerText = 'Edit Transfer';
                    document.getElementById('transferProduct').value = t.productId;
                    document.getElementById('transferFrom').value = t.fromLocationId;
                    document.getElementById('transferFromStock').value = t.quantity;
                    document.getElementById('transferTo').value = t.toLocationId;
                    document.getElementById('transferQuantity').value = t.quantity;
                    document.getElementById('transferModal').classList.remove('hidden');
                });
            });
        }

        document.getElementById('newTransferBtn').addEventListener('click', () => {
            window._editTransferIndex = null;
            document.getElementById('transferModalTitle').innerText = 'New Transfer';
            document.getElementById('transferProduct').value = 'Select Product';
            document.getElementById('transferFrom').value = 'Select Location';
            document.getElementById('transferFromStock').value = '';
            document.getElementById('transferTo').value = 'Select Location';
            document.getElementById('transferQuantity').value = '';

            const prodSelect = document.getElementById('transferProduct');
            prodSelect.innerHTML = '<option>Select Product</option>' +
                products.map(p => `<option value="${p.id}">${p.name}</option>`).join('');

            const fromSelect = document.getElementById('transferFrom');
            fromSelect.innerHTML = '<option>Select Location</option>' +
                locations.map(l => `<option value="${l.id}">${l.name} (${l.city})</option>`).join('');

            const toSelect = document.getElementById('transferTo');
            toSelect.innerHTML = '<option>Select Location</option>' +
                locations.map(l => `<option value="${l.id}">${l.name} (${l.city})</option>`).join('');

            document.getElementById('transferModal').classList.remove('hidden');
        });

        document.getElementById('transferProduct').addEventListener('change', () => {
            const prodId = document.getElementById('transferProduct').value;
            const fromLocId = document.getElementById('transferFrom').value;

            if (prodId && prodId !== 'Select Product' && fromLocId && fromLocId !== 'Select Location') {
                const stock = inventory[prodId]?.[fromLocId] || 0;
                document.getElementById('transferFromStock').value = stock;
            }
        });

        document.getElementById('transferFrom').addEventListener('change', () => {
            const prodId = document.getElementById('transferProduct').value;
            const fromLocId = document.getElementById('transferFrom').value;

            if (prodId && prodId !== 'Select Product' && fromLocId && fromLocId !== 'Select Location') {
                const stock = inventory[prodId]?.[fromLocId] || 0;
                document.getElementById('transferFromStock').value = stock;
            }
        });

        document.getElementById('saveTransferBtn').addEventListener('click', () => {
            const prodId = document.getElementById('transferProduct').value;
            const fromLocId = document.getElementById('transferFrom').value;
            const toLocId = document.getElementById('transferTo').value;
            const qty = parseInt(document.getElementById('transferQuantity').value) || 0;

            if (!prodId || prodId === 'Select Product' || !fromLocId || fromLocId === 'Select Location' ||
                !toLocId || toLocId === 'Select Location' || !qty || qty <= 0) {
                alert('Complete all fields');
                return;
            }

            if (fromLocId === toLocId) {
                alert('From and To locations must be different');
                return;
            }

            const availableStock = inventory[prodId]?.[fromLocId] || 0;

            // Editing existing transfer
            if (window._editTransferIndex !== null && window._editTransferIndex !== undefined) {
                const oldTransfer = transfers[window._editTransferIndex];
                // Reverse old transfer
                inventory[oldTransfer.productId][oldTransfer.fromLocationId] += oldTransfer.quantity;
                inventory[oldTransfer.productId][oldTransfer.toLocationId] -= oldTransfer.quantity;

                // Check new qty
                const newAvailable = inventory[prodId][fromLocId];
                if (qty > newAvailable) {
                    alert(`Not enough stock! Available: ${newAvailable}`);
                    return;
                }

                // Apply new transfer
                inventory[prodId][fromLocId] -= qty;
                inventory[prodId][toLocId] = (inventory[prodId][toLocId] || 0) + qty;

                const prod = products.find(p => p.id === prodId);
                const fromLoc = locations.find(l => l.id === fromLocId);
                const toLoc = locations.find(l => l.id === toLocId);

                transfers[window._editTransferIndex] = {
                    id: oldTransfer.id,
                    productId: prodId,
                    productName: prod.name,
                    fromLocationId: fromLocId,
                    fromLocation: fromLoc.name,
                    toLocationId: toLocId,
                    toLocation: toLoc.name,
                    quantity: qty,
                    date: oldTransfer.date
                };
            } else {
                // New transfer
                if (qty > availableStock) {
                    alert(`Not enough stock! Available: ${availableStock}`);
                    return;
                }

                inventory[prodId][fromLocId] -= qty;
                inventory[prodId][toLocId] = (inventory[prodId][toLocId] || 0) + qty;

                const prod = products.find(p => p.id === prodId);
                const fromLoc = locations.find(l => l.id === fromLocId);
                const toLoc = locations.find(l => l.id === toLocId);

                transfers.push({
                    id: Date.now().toString(),
                    productId: prodId,
                    productName: prod.name,
                    fromLocationId: fromLocId,
                    fromLocation: fromLoc.name,
                    toLocationId: toLocId,
                    toLocation: toLoc.name,
                    quantity: qty,
                    date: new Date().toISOString()
                });
            }

            saveData();
            document.getElementById('transferModal').classList.add('hidden');
        });

        document.getElementById('closeTransferModal').addEventListener('click', () => {
            document.getElementById('transferModal').classList.add('hidden');
        });

        document.getElementById('searchTransfer').addEventListener('input', renderTransfers);

        // ============ EXPORT & CLEAR ============
        document.getElementById('exportProductsBtn').addEventListener('click', () => {
            const data = [['Location', ...products.map(p => p.name)]];
            locations.forEach(loc => {
                const row = [loc.name];
                products.forEach(prod => {
                    row.push(inventory[prod.id]?.[loc.id] || 0);
                });
                data.push(row);
            });

            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Inventory');
            XLSX.writeFile(wb, 'stockflow_inventory.xlsx');
        });

        document.getElementById('clearAllBtn').addEventListener('click', () => {
            if (confirm('Clear all data?')) {
                locations = [];
                products = [];
                inventory = {};
                deliveries = [];
                transfers = [];
                saveData();
            }
        });

        // ============ INIT ============
        updateDashboard();
    </script>
</body>
</html>
