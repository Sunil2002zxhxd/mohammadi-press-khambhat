<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Printing Press Order Manager - Mohammadi</title>
  <style>
    body{font-family: Arial, Helvetica, sans-serif; margin:16px; background:#f7f7f9;}
    h1{font-size:20px;margin:0 0 8px;}
    .card{background:#fff;padding:12px;border-radius:8px;box-shadow:0 1px 3px rgba(0,0,0,0.08);margin-bottom:12px;}
    label{display:block;margin:6px 0 2px;font-weight:600;}
    input,select,textarea{width:100%;padding:8px;border:1px solid #ddd;border-radius:6px;box-sizing:border-box;}
    button{padding:8px 12px;border-radius:6px;border:0;background:#0b79d0;color:#fff;cursor:pointer;}
    table{width:100%;border-collapse:collapse;margin-top:8px;}
    th,td{padding:8px;border-bottom:1px solid #eee;text-align:left;font-size:13px;}
    .muted{color:#666;font-size:13px;}
    .status-btn{margin-right:6px;padding:5px 8px;border-radius:5px;border:0;background:#6c757d;color:#fff;}
    .primary{background:#0b79d0;}
    .danger{background:#d9534f;}
    .small{font-size:12px;padding:4px 6px;}
    .flex{display:flex;gap:8px;align-items:center;}
    .top-actions{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px;}
    .tag{display:inline-block;padding:4px 8px;border-radius:12px;background:#eef4fb;color:#0b79d0;font-weight:600;font-size:12px;}
    .search{width:240px;}
    @media (min-width:900px){ .grid{display:grid;grid-template-columns:360px 1fr;gap:12px;} }
  </style>
</head>
<body>
  <h1>Printing Press Order Manager — Mohammadi</h1>
  <div class="muted">Order → Design → Printing → Finishing → Delivery. Browser local storage में डेटा सुरक्षित रहेगा.</div>

  <div class="grid" style="margin-top:12px;">
    <!-- Left: Form -->
    <div class="card">
      <strong>नया Order जोड़ें / New Order</strong>
      <div style="margin-top:8px;">
        <label>कस्टमर का नाम / Customer</label>
        <input id="custName" placeholder="नाम लिखें / e.g. Ramesh Patel" />

        <label>फोन / Phone</label>
        <input id="custPhone" placeholder="Mobile number" />

        <label>Job Description (e.g. kankotri 500 pcs)</label>
        <input id="jobDesc" placeholder="Job details" />

        <label>Paper / Size / Finish</label>
        <input id="paperSpec" placeholder="Paper type, size, finish" />

        <label>Quantity</label>
        <input id="qty" type="number" min="1" value="1" />

        <label>Price (INR)</label>
        <input id="price" type="number" min="0" value="0" />

        <label>Notes / Internal</label>
        <textarea id="notes" rows="3" placeholder="Proof details, delivery address, etc."></textarea>

        <div style="margin-top:8px;" class="flex">
          <button id="addOrderBtn" class="primary">Add Order</button>
          <button id="clearForm" class="small">Clear</button>
        </div>
      </div>

      <hr style="margin:12px 0" />
      <div>
        <strong>Import / Export</strong>
        <div style="margin-top:8px;" class="flex">
          <button id="exportCsv">Export CSV</button>
          <button id="exportJson">Export JSON</button>
          <input id="importFile" type="file" accept=".json" style="margin-left:8px"/>
        </div>
        <div class="muted" style="margin-top:8px;font-size:13px;">
          Export CSV से आप Excel में खोल सकते हैं. JSON import केवल वही फॉर्मेट स्वीकार करेगा जो यहाँ export करता है.
        </div>
      </div>
    </div>

    <!-- Right: Orders list -->
    <div>
      <div class="card top-actions">
        <div class="flex">
          <input id="search" class="search" placeholder="Search name / job / phone" />
          <select id="filterStatus">
            <option value="">All Status</option>
            <option value="Received">Received</option>
            <option value="Design">Design</option>
            <option value="Printing">Printing</option>
            <option value="Finishing">Finishing</option>
            <option value="Packed">Packed</option>
            <option value="Out for delivery">Out for delivery</option>
            <option value="Delivered">Delivered</option>
            <option value="Cancelled">Cancelled</option>
          </select>
        </div>
        <div style="margin-left:auto" class="flex">
          <div class="tag" id="totalOrders">0 Orders</div>
          <button id="deleteAll" class="danger small">Delete All</button>
        </div>
      </div>

      <div id="ordersCard" class="card">
        <table id="ordersTable">
          <thead>
            <tr><th>#</th><th>Customer</th><th>Job</th><th>Qty</th><th>Price</th><th>Status</th><th>Actions</th></tr>
          </thead>
          <tbody id="ordersBody"></tbody>
        </table>
      </div>
    </div>
  </div>

<script>
/*
 Simple order manager using localStorage.
 Order object:
 { id, custName, custPhone, jobDesc, paperSpec, qty, price, notes, status, createdAt }
 Status flow: Received -> Design -> Printing -> Finishing -> Packed -> Out for delivery -> Delivered
*/

const STATUS_FLOW = ['Received','Design','Printing','Finishing','Packed','Out for delivery','Delivered'];
const LS_KEY = 'printing_orders_mohammadi_v1';

function uid(){
  return 'o'+Date.now().toString(36)+'_'+Math.random().toString(36).slice(2,7);
}

function loadOrders(){
  const raw = localStorage.getItem(LS_KEY);
  return raw ? JSON.parse(raw) : [];
}

function saveOrders(arr){
  localStorage.setItem(LS_KEY, JSON.stringify(arr));
}

function addOrder(o){
  const arr = loadOrders();
  arr.unshift(o);
  saveOrders(arr);
  renderOrders();
}

function updateOrder(id, changes){
  const arr = loadOrders();
  const ix = arr.findIndex(x => x.id === id);
  if(ix === -1) return;
  arr[ix] = {...arr[ix], ...changes};
  saveOrders(arr);
  renderOrders();
}

function deleteOrder(id){
  let arr = loadOrders();
  arr = arr.filter(x => x.id !== id);
  saveOrders(arr);
  renderOrders();
}

function clearAll(){
  if(!confirm('Confirm delete ALL orders?')) return;
  localStorage.removeItem(LS_KEY);
  renderOrders();
}

function formatINR(n){ return '₹'+Number(n||0).toLocaleString('en-IN'); }

function renderOrders(){
  const body = document.getElementById('ordersBody');
  const totalTag = document.getElementById('totalOrders');
  body.innerHTML = '';
  const arr = loadOrders();
  // apply search + filter
  const q = document.getElementById('search').value.trim().toLowerCase();
  const st = document.getElementById('filterStatus').value;
  const filtered = arr.filter(o=>{
    const matchQ = q === '' || [o.custName,o.custPhone,o.jobDesc,o.paperSpec].join(' ').toLowerCase().includes(q);
    const matchS = st === '' || o.status === st;
    return matchQ && matchS;
  });

  filtered.forEach((o, idx) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${idx+1}</td>
      <td><strong>${o.custName}</strong><div class="muted">${o.custPhone||''}</div></td>
      <td>${o.jobDesc}<div class="muted">${o.paperSpec||''}</div></td>
      <td>${o.qty}</td>
      <td>${formatINR(o.price)}</td>
      <td>${o.status}<div class="muted" style="font-size:12px">Created: ${new Date(o.createdAt).toLocaleString()}</div></td>
      <td>
        <div style="display:flex;gap:6px;flex-wrap:wrap;">
          <button class="small" onclick="advanceStatus('${o.id}')">Next</button>
          <button class="small" onclick="openEdit('${o.id}')">Edit</button>
          <button class="small" onclick="markCancelled('${o.id}')">Cancel</button>
          <button class="small" onclick="deleteOrder('${o.id}')">Delete</button>
        </div>
      </td>
    `;
    body.appendChild(tr);
  });

  totalTag.textContent = filtered.length + ' Orders';
}

// Form handling
document.getElementById('addOrderBtn').addEventListener('click', ()=>{
  const custName = document.getElementById('custName').value.trim();
  const custPhone = document.getElementById('custPhone').value.trim();
  const jobDesc = document.getElementById('jobDesc').value.trim();
  const paperSpec = document.getElementById('paperSpec').value.trim();
  const qty = Number(document.getElementById('qty').value || 0);
  const price = Number(document.getElementById('price').value || 0);
  const notes = document.getElementById('notes').value.trim();

  if(!custName || !jobDesc || qty<=0){
    alert('कुछ जरूरी fields भरें: Customer name, Job description, Quantity.');
    return;
  }

  const order = {
    id: uid(),
    custName, custPhone, jobDesc, paperSpec, qty, price, notes,
    status: 'Received',
    createdAt: new Date().toISOString()
  };
  addOrder(order);
  document.getElementById('clearForm').click();
});

// clear form
document.getElementById('clearForm').addEventListener('click', ()=>{
  ['custName','custPhone','jobDesc','paperSpec','qty','price','notes'].forEach(id=>{
    const el = document.getElementById(id);
    if(el.type === 'number') el.value = id==='qty' ? 1 : 0;
    else el.value = '';
  });
});

// search & filter
document.getElementById('search').addEventListener('input', renderOrders);
document.getElementById('filterStatus').addEventListener('change', renderOrders);

// advance status
function advanceStatus(id){
  const arr = loadOrders();
  const o = arr.find(x=>x.id===id);
  if(!o) return;
  if(o.status === 'Delivered') { alert('Already Delivered'); return;}
  if(o.status === 'Cancelled') { alert('Order is Cancelled'); return;}
  const curIndex = STATUS_FLOW.indexOf(o.status);
  const next = STATUS_FLOW[Math.min(curIndex+1, STATUS_FLOW.length-1)];
  updateOrder(id, { status: next });
}

// quick mark cancelled
function markCancelled(id){
  if(!confirm('Mark this order as Cancelled?')) return;
  updateOrder(id, { status: 'Cancelled' });
}

// edit: open a prompt sequence (simple)
function openEdit(id){
  const arr = loadOrders();
  const o = arr.find(x=>x.id===id);
  if(!o) return;
  const name = prompt('Customer name', o.custName);
  if(name === null) return;
  const phone = prompt('Phone', o.custPhone || '');
  if(phone === null) return;
  const job = prompt('Job description', o.jobDesc);
  if(job === null) return;
  const qty = prompt('Quantity', o.qty);
  if(qty === null) return;
  const price = prompt('Price (INR)', o.price);
  if(price === null) return;
  updateOrder(id, { custName: name, custPhone: phone, jobDesc: job, qty: Number(qty), price: Number(price) });
}

// export CSV
function exportCSV(){
  const arr = loadOrders();
  if(arr.length===0){ alert('No orders'); return; }
  const headers = ['id','custName','custPhone','jobDesc','paperSpec','qty','price','status','createdAt','notes'];
  const rows = arr.map(a => headers.map(h => `"${String(a[h]||'').replace(/"/g,'""')}"`).join(','));
  const csv = [headers.join(','), ...rows].join('\n');
  const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'printing_orders_export.csv'; a.click();
  URL.revokeObjectURL(url);
}

// export JSON
function exportJSON(){
  const arr = loadOrders();
  const blob = new Blob([JSON.stringify(arr, null, 2)], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'printing_orders_export.json'; a.click();
  URL.revokeObjectURL(url);
}

// import JSON
document.getElementById('importFile').addEventListener('change', (ev)=>{
  const f = ev.target.files[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = function(e){
    try{
      const data = JSON.parse(e.target.result);
      if(!Array.isArray(data)){ alert('Invalid JSON format'); return; }
      // simple merge: prepend imported
      const current = loadOrders();
      const merged = [...data, ...current];
      saveOrders(merged);
      renderOrders();
      alert('Imported '+data.length+' records.');
    }catch(err){
      alert('Error parsing file: '+err);
    }
  };
  reader.readAsText(f);
});

// delete all
document.getElementById('deleteAll').addEventListener('click', clearAll);

// attach export handlers
document.getElementById('exportCsv').addEventListener('click', exportCSV);
document.getElementById('exportJson').addEventListener('click', exportJSON);

// on load
renderOrders();

// expose deleteOrder to global (for inline onclick)
window.deleteOrder = deleteOrder;
window.advanceStatus = advanceStatus;
window.openEdit = openEdit;
window.markCancelled = markCancelled;
</script>
</body>
</html>
