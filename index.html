<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StockFlow - Inventory App</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background: #f0f0f0; padding: 20px; }
        .container { max-width: 1200px; margin: 0 auto; }
        header { background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
        header h1 { color: #0066cc; margin-bottom: 10px; }
        .tabs { display: flex; gap: 10px; margin-bottom: 20px; }
        .tabs button { padding: 10px 20px; background: white; border: 2px solid #ccc; border-radius: 5px; cursor: pointer; font-size: 16px; }
        .tabs button.active { background: #0066cc; color: white; border-color: #0066cc; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; margin-bottom: 15px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
        .grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin-bottom: 20px; }
        .stat { background: white; padding: 20px; border-radius: 8px; text-align: center; }
        .stat-number { font-size: 32px; font-weight: bold; color: #0066cc; }
        .stat-label { color: #666; margin-top: 5px; }
        input, select { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; }
        button { padding: 10px 20px; background: #0066cc; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; margin-top: 10px; }
        button.danger { background: #cc3333; }
        button.success { background: #00aa00; }
        .product-item, .delivery-item { background: #f9f9f9; padding: 15px; border-radius: 5px; margin-bottom: 10px; border-left: 4px solid #0066cc; }
        .product-actions { display: flex; gap: 10px; margin-top: 10px; }
        .product-actions button { flex: 1; margin: 0; padding: 8px; font-size: 14px; }
        .low-stock { border-left-color: #ff6600; background: #fff3e0; }
        .modal { display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); z-index: 1000; }
        .modal.active { display: flex; align-items: center; justify-content: center; }
        .modal-content { background: white; padding: 30px; border-radius: 8px; width: 90%; max-width: 500px; }
        .modal-content h2 { margin-bottom: 20px; color: #0066cc; }
        .modal-buttons { display: flex; gap: 10px; margin-top: 20px; }
        .modal-buttons button { flex: 1; }
        h3 { color: #0066cc; margin: 15px 0; }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📦 StockFlow - Inventory Management</h1>
            <p>Track products and deliveries - No setup required!</p>
        </header>

        <!-- Tabs -->
        <div class="tabs">
            <button class="tab-btn active" data-tab="dashboard">📊 Dashboard</button>
            <button class="tab-btn" data-tab="products">📦 Products</button>
            <button class="tab-btn" data-tab="deliveries">🚚 Deliveries</button>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboard" class="tab-content active">
            <div class="grid">
                <div class="stat">
                    <div class="stat-number" id="totalProductsNum">0</div>
                    <div class="stat-label">Total Products</div>
                </div>
                <div class="stat">
                    <div class="stat-number" id="lowStockNum">0</div>
                    <div class="stat-label">Low Stock</div>
                </div>
                <div class="stat">
                    <div class="stat-number" id="totalItemsNum">0</div>
                    <div class="stat-label">Total Items</div>
                </div>
                <div class="stat">
                    <div class="stat-number" id="deliveriesNum">0</div>
                    <div class="stat-label">Deliveries</div>
                </div>
            </div>
            <div class="card">
                <h3>⚠️ Low Stock Alerts</h3>
                <div id="lowStockAlerts">No low stock items</div>
            </div>
        </div>

        <!-- Products Tab -->
        <div id="products" class="tab-content">
            <button class="success" onclick="openProductModal()">➕ Add New Product</button>
            <input type="text" id="searchProducts" placeholder="🔍 Search products..." style="margin-top: 15px;">
            <button onclick="exportProducts()" style="background: #666; margin-left: 10px;">📥 Export</button>
            <button onclick="clearAll()" class="danger" style="margin-left: 10px;">🗑️ Clear All</button>
            <div id="productsList" style="margin-top: 20px;"></div>
        </div>

        <!-- Deliveries Tab -->
        <div id="deliveries" class="tab-content">
            <button class="success" onclick="openDeliveryModal()">🚚 New Delivery</button>
            <div id="deliveriesList" style="margin-top: 20px;"></div>
        </div>
    </div>

    <!-- Product Modal -->
    <div id="productModal" class="modal">
        <div class="modal-content">
            <h2>Add Product</h2>
            <input type="text" id="productName" placeholder="Product Name" required>
            <input type="text" id="productSku" placeholder="SKU/Code" required>
            <input type="number" id="productQty" placeholder="Quantity" value="0" required>
            <input type="number" id="productPrice" placeholder="Price" value="0" step="0.01" required>
            <input type="number" id="productThreshold" placeholder="Low Stock Threshold" value="10" required>
            <div class="modal-buttons">
                <button onclick="saveProduct()">Save</button>
                <button class="danger" onclick="closeModal('productModal')">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Delivery Modal -->
    <div id="deliveryModal" class="modal">
        <div class="modal-content">
            <h2>Create Delivery</h2>
            <select id="deliveryProduct" required>
                <option>Select Product</option>
            </select>
            <input type="number" id="deliveryQty" placeholder="Quantity" value="1" required>
            <input type="text" id="customerName" placeholder="Customer Name" required>
            <input type="text" id="customerAddress" placeholder="Address" required>
            <select id="deliveryStatus">
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
                <option value="failed">Failed</option>
            </select>
            <div class="modal-buttons">
                <button onclick="saveDelivery()">Save</button>
                <button class="danger" onclick="closeModal('deliveryModal')">Cancel</button>
            </div>
        </div>
    </div>

    <script>
        // Data Storage
        let products = JSON.parse(localStorage.getItem('sf_products')) || [];
        let deliveries = JSON.parse(localStorage.getItem('sf_deliveries')) || [];

        // Save Data
        function save() {
            localStorage.setItem('sf_products', JSON.stringify(products));
            localStorage.setItem('sf_deliveries', JSON.stringify(deliveries));
            render();
        }

        // Tab Switching
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
                btn.classList.add('active');
                document.getElementById(btn.dataset.tab).classList.add('active');
            });
        });

        // Update Dashboard
        function updateDashboard() {
            document.getElementById('totalProductsNum').textContent = products.length;
            const lowStock = products.filter(p => p.qty <= p.threshold).length;
            document.getElementById('lowStockNum').textContent = lowStock;
            document.getElementById('totalItemsNum').textContent = products.reduce((sum, p) => sum + p.qty, 0);
            document.getElementById('deliveriesNum').textContent = deliveries.length;

            const alerts = products.filter(p => p.qty <= p.threshold);
            if (alerts.length === 0) {
                document.getElementById('lowStockAlerts').innerHTML = '<p>✅ All stock levels healthy</p>';
            } else {
                document.getElementById('lowStockAlerts').innerHTML = alerts.map(p => 
                    `<div style="padding: 10px; background: #ffe0e0; border-radius: 5px; margin-bottom: 10px;">
                        <strong>${p.name}</strong> - Stock: ${p.qty}/${p.threshold}
                    </div>`
                ).join('');
            }
        }

        // Render Products
        function renderProducts() {
            const search = document.getElementById('searchProducts').value.toLowerCase();
            const filtered = products.filter(p => p.name.toLowerCase().includes(search) || p.sku.toLowerCase().includes(search));
            const html = filtered.map(p => `
                <div class="product-item ${p.qty <= p.threshold ? 'low-stock' : ''}">
                    <h4>${p.name}</h4>
                    <p>SKU: ${p.sku} | Stock: <strong>${p.qty}</strong> | Price: $${p.price.toFixed(2)}</p>
                    <div class="product-actions">
                        <button onclick="editProduct('${p.id}')">✏️ Edit</button>
                        <button onclick="addStock('${p.id}')" class="success">➕ Add</button>
                        <button onclick="removeStock('${p.id}')">➖ Remove</button>
                        <button onclick="deleteProduct('${p.id}')" class="danger">🗑️ Delete</button>
                    </div>
                </div>
            `).join('');
            document.getElementById('productsList').innerHTML = html || '<p>No products. Add one to get started!</p>';
        }

        // Render Deliveries
        function renderDeliveries() {
            const html = deliveries.map(d => `
                <div class="delivery-item">
                    <h4>${d.product} × ${d.qty}</h4>
                    <p>Customer: <strong>${d.customer}</strong></p>
                    <p>Address: ${d.address}</p>
                    <p>Status: <select onchange="updateDeliveryStatus('${d.id}', this.value)">
                        <option value="pending" ${d.status === 'pending' ? 'selected' : ''}>Pending</option>
                        <option value="completed" ${d.status === 'completed' ? 'selected' : ''}>Completed</option>
                        <option value="failed" ${d.status === 'failed' ? 'selected' : ''}>Failed</option>
                    </select></p>
                    <button onclick="deleteDelivery('${d.id}')" class="danger">Delete</button>
                </div>
            `).join('');
            document.getElementById('deliveriesList').innerHTML = html || '<p>No deliveries yet.</p>';
        }

        // Render All
        function render() {
            updateDashboard();
            renderProducts();
            renderDeliveries();
        }

        // Product Functions
        function openProductModal() {
            document.getElementById('productName').value = '';
            document.getElementById('productSku').value = '';
            document.getElementById('productQty').value = '0';
            document.getElementById('productPrice').value = '0';
            document.getElementById('productThreshold').value = '10';
            window.editingId = null;
            document.getElementById('productModal').classList.add('active');
        }

        function saveProduct() {
            const name = document.getElementById('productName').value.trim();
            const sku = document.getElementById('productSku').value.trim();
            const qty = parseInt(document.getElementById('productQty').value) || 0;
            const price = parseFloat(document.getElementById('productPrice').value) || 0;
            const threshold = parseInt(document.getElementById('productThreshold').value) || 10;

            if (!name || !sku) { alert('Name and SKU required'); return; }

            if (window.editingId) {
                const p = products.find(x => x.id === window.editingId);
                if (p) { p.name = name; p.sku = sku; p.qty = qty; p.price = price; p.threshold = threshold; }
            } else {
                products.push({ id: Date.now().toString(), name, sku, qty, price, threshold });
            }
            save();
            closeModal('productModal');
        }

        function editProduct(id) {
            const p = products.find(x => x.id === id);
            if (p) {
                document.getElementById('productName').value = p.name;
                document.getElementById('productSku').value = p.sku;
                document.getElementById('productQty').value = p.qty;
                document.getElementById('productPrice').value = p.price;
                document.getElementById('productThreshold').value = p.threshold;
                window.editingId = id;
                document.getElementById('productModal').classList.add('active');
            }
        }

        function deleteProduct(id) {
            if (confirm('Delete this product?')) {
                products = products.filter(p => p.id !== id);
                save();
            }
        }

        function addStock(id) {
            const p = products.find(x => x.id === id);
            if (p) { p.qty++; save(); }
        }

        function removeStock(id) {
            const p = products.find(x => x.id === id);
            if (p && p.qty > 0) { p.qty--; save(); }
        }

        // Delivery Functions
        function openDeliveryModal() {
            const select = document.getElementById('deliveryProduct');
            select.innerHTML = '<option>Select Product</option>' + products.map(p => `<option value="${p.id}">${p.name} (${p.qty} in stock)</option>`).join('');
            document.getElementById('deliveryQty').value = '1';
            document.getElementById('customerName').value = '';
            document.getElementById('customerAddress').value = '';
            document.getElementById('deliveryStatus').value = 'pending';
            document.getElementById('deliveryModal').classList.add('active');
        }

        function saveDelivery() {
            const productId = document.getElementById('deliveryProduct').value;
            const qty = parseInt(document.getElementById('deliveryQty').value) || 1;
            const customer = document.getElementById('customerName').value.trim();
            const address = document.getElementById('customerAddress').value.trim();
            const status = document.getElementById('deliveryStatus').value;

            if (productId === 'Select Product' || !customer) { alert('All fields required'); return; }

            const product = products.find(p => p.id === productId);
            if (!product) { alert('Product not found'); return; }
            if (product.qty < qty) { alert('Not enough stock'); return; }

            product.qty -= qty;
            deliveries.push({
                id: Date.now().toString(),
                product: product.name,
                qty,
                customer,
                address,
                status,
                date: new Date().toISOString()
            });
            save();
            closeModal('deliveryModal');
        }

        function updateDeliveryStatus(id, status) {
            const d = deliveries.find(x => x.id === id);
            if (d) { d.status = status; save(); }
        }

        function deleteDelivery(id) {
            if (confirm('Delete delivery?')) {
                deliveries = deliveries.filter(d => d.id !== id);
                save();
            }
        }

        // Utilities
        function closeModal(id) {
            document.getElementById(id).classList.remove('active');
        }

        function exportProducts() {
            if (products.length === 0) { alert('No products to export'); return; }
            let csv = 'Product Name,SKU,Quantity,Price,Threshold\n';
            csv += products.map(p => `"${p.name}","${p.sku}",${p.qty},${p.price},${p.threshold}`).join('\n');
            const blob = new Blob([csv], { type: 'text/csv' });
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'stockflow-products.csv';
            a.click();
        }

        function clearAll() {
            if (confirm('⚠️ Delete ALL data? This cannot be undone!')) {
                products = [];
                deliveries = [];
                save();
            }
        }

        // Search
        document.getElementById('searchProducts').addEventListener('input', renderProducts);

        // Init
        render();
    </script>
</body>
</html>
