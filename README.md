<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>AK Cozy and Co. — Order Tracker</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet"/>

<!-- Firebase Realtime Database (compat SDK — no module syntax needed) -->
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-database-compat.js"></script>

<style>
  :root {
    --lav50:#f7f4fd;--lav100:#ede8fa;--lav200:#ddd4f6;
    --lav300:#c7b8f0;--lav400:#b09de8;--lav500:#9a82de;--lav600:#8469cc;
    --text-dark:#3d3356;--text-mid:#6b5f8a;--text-soft:#9a90b4;
    --white:#ffffff;--surface:#faf8ff;
  }
  *,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
  body{font-family:'DM Sans',sans-serif;background:var(--lav50);color:var(--text-dark);min-height:100vh;}
  input,select,textarea{font-family:'DM Sans',sans-serif;}
  input:focus,select:focus,textarea:focus{border-color:var(--lav400)!important;outline:none;}

  /* Header */
  header{background:linear-gradient(135deg,var(--lav200),var(--lav300));padding:24px 36px;
    display:flex;align-items:center;justify-content:space-between;flex-wrap:wrap;gap:14px;}
  .brand{display:flex;align-items:center;gap:12px;}
  .brand-icon{width:46px;height:46px;background:var(--white);border-radius:50%;
    display:flex;align-items:center;justify-content:center;font-size:22px;
    box-shadow:0 2px 8px rgba(148,120,210,.15);}
  .brand h1{font-family:'Playfair Display',Georgia,serif;font-size:1.45rem;color:var(--text-dark);line-height:1.2;}
  .brand p{font-size:.7rem;color:var(--text-mid);letter-spacing:.07em;text-transform:uppercase;font-weight:500;}
  .header-right{display:flex;gap:14px;align-items:center;flex-wrap:wrap;}
  #sync-status{font-size:.75rem;font-weight:500;color:var(--text-mid);letter-spacing:.04em;}
  .pill{background:rgba(255,255,255,.65);border-radius:50px;padding:8px 18px;text-align:center;
    backdrop-filter:blur(4px);border:1px solid rgba(255,255,255,.8);}
  .pill .val{font-family:'Playfair Display',Georgia,serif;font-size:1.15rem;color:var(--text-dark);display:block;}
  .pill .lbl{font-size:.66rem;color:var(--text-mid);text-transform:uppercase;letter-spacing:.07em;font-weight:500;}

  /* Main */
  main{padding:30px 36px;max-width:1120px;margin:0 auto;}

  /* Month nav */
  .month-nav{display:flex;align-items:center;gap:10px;margin-bottom:24px;flex-wrap:wrap;}
  #month-label{font-family:'Playfair Display',Georgia,serif;font-size:1.55rem;color:var(--text-dark);flex:1;}
  .nav-btn{background:var(--white);border:1.5px solid var(--lav300);color:var(--text-mid);
    border-radius:8px;width:38px;height:38px;cursor:pointer;font-size:1.1rem;
    display:flex;align-items:center;justify-content:center;transition:background .18s;font-family:'DM Sans',sans-serif;}
  .nav-btn:hover{background:var(--lav100);}
  .today-btn{width:auto!important;padding:0 14px!important;font-size:.76rem!important;font-weight:500;}

  /* Revenue banner */
  .revenue-banner{background:linear-gradient(120deg,var(--lav300),var(--lav400));
    border-radius:22px;padding:22px 30px;display:flex;align-items:center;
    justify-content:space-between;flex-wrap:wrap;gap:16px;margin-bottom:24px;
    box-shadow:0 4px 20px rgba(148,120,210,.18);}
  .rev-lbl{font-size:.68rem;text-transform:uppercase;letter-spacing:.1em;color:rgba(255,255,255,.85);font-weight:500;}
  .rev-num{font-family:'Playfair Display',Georgia,serif;font-size:2.3rem;color:#fff;line-height:1.1;}
  .rev-breakdown{display:flex;gap:24px;flex-wrap:wrap;}
  .rev-stat{text-align:center;}
  .rs-val{font-size:1.15rem;font-weight:500;color:#fff;}
  .rs-lbl{font-size:.66rem;color:rgba(255,255,255,.8);text-transform:uppercase;letter-spacing:.07em;}

  /* Toolbar */
  .toolbar{display:flex;justify-content:space-between;align-items:center;margin-bottom:18px;gap:12px;flex-wrap:wrap;}
  #search-input{border:1.5px solid var(--lav200);border-radius:8px;padding:9px 12px;
    font-size:.88rem;color:var(--text-dark);background:var(--white);max-width:300px;flex:1;min-width:160px;}
  .btn-add{background:var(--lav500);color:#fff;border:none;border-radius:8px;padding:10px 22px;
    font-size:.88rem;font-weight:500;cursor:pointer;box-shadow:0 3px 12px rgba(154,130,222,.35);
    white-space:nowrap;transition:background .18s;font-family:'DM Sans',sans-serif;}
  .btn-add:hover{background:var(--lav600);}

  /* Table */
  .table-wrap{background:var(--white);border-radius:20px;
    box-shadow:0 2px 8px rgba(148,120,210,.10);border:1px solid var(--lav100);overflow:hidden;}
  .table-scroll{overflow-x:auto;}
  table{width:100%;border-collapse:collapse;}
  thead tr{background:var(--lav100);}
  th{text-align:left;padding:13px 16px;font-size:.7rem;text-transform:uppercase;
    letter-spacing:.09em;color:var(--text-mid);font-weight:600;white-space:nowrap;
    cursor:pointer;user-select:none;}
  th:hover{color:var(--lav600);}
  td{padding:13px 16px;font-size:.88rem;color:var(--text-dark);
    border-bottom:1px solid var(--lav50);vertical-align:middle;}
  tbody tr:last-child td{border-bottom:none;}
  tbody tr:hover td{background:var(--lav50);}
  .action-btn{background:none;border:none;cursor:pointer;font-size:1rem;
    padding:3px 6px;border-radius:6px;color:var(--text-soft);}
  #empty-state{text-align:center;padding:60px 20px;color:var(--text-soft);font-size:.9rem;display:none;}

  /* Export */
  .export-row{text-align:right;margin-top:12px;}
  .btn-export{background:none;border:1px solid var(--lav300);color:var(--text-mid);
    border-radius:8px;padding:6px 14px;font-size:.76rem;cursor:pointer;font-family:'DM Sans',sans-serif;}
  .btn-export:hover{background:var(--lav100);}

  /* Modal overlay */
  .modal-overlay{display:none;position:fixed;inset:0;background:rgba(61,51,86,.38);
    backdrop-filter:blur(3px);z-index:100;align-items:center;justify-content:center;padding:16px;}
  .modal-overlay.open{display:flex;}
  .modal{background:var(--white);border-radius:22px;padding:36px 30px;width:100%;max-width:480px;
    box-shadow:0 12px 40px rgba(100,80,160,.2);position:relative;
    max-height:90vh;overflow-y:auto;animation:slideUp .22s ease;}
  @keyframes slideUp{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:translateY(0)}}
  .modal-close{position:absolute;top:16px;right:18px;background:none;border:none;
    cursor:pointer;font-size:1.2rem;color:var(--text-soft);}
  .modal h2{font-family:'Playfair Display',Georgia,serif;font-size:1.3rem;color:var(--text-dark);margin-bottom:24px;}
  .form-grid{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:14px;}
  .form-row{margin-bottom:14px;}
  .form-row label{display:block;font-size:.7rem;font-weight:600;text-transform:uppercase;
    letter-spacing:.08em;color:var(--text-mid);margin-bottom:5px;}
  .form-row input,.form-row select,.form-row textarea{width:100%;border:1.5px solid var(--lav200);
    border-radius:8px;padding:9px 11px;font-size:.88rem;color:var(--text-dark);background:var(--surface);}
  .form-row textarea{resize:vertical;min-height:72px;}
  #form-error{color:#c06060;font-size:.8rem;min-height:20px;margin-top:4px;}
  .modal-footer{display:flex;gap:10px;justify-content:flex-end;margin-top:20px;}
  .btn-cancel{background:var(--lav100);color:var(--text-mid);border:none;border-radius:8px;
    padding:10px 20px;cursor:pointer;font-family:'DM Sans',sans-serif;font-size:.87rem;font-weight:500;}
  .btn-save{background:var(--lav500);color:#fff;border:none;border-radius:8px;padding:10px 24px;
    cursor:pointer;font-family:'DM Sans',sans-serif;font-size:.87rem;font-weight:500;
    box-shadow:0 3px 10px rgba(154,130,222,.3);}
  .btn-save:hover{background:var(--lav600);}

  /* Confirm delete */
  .confirm-box{background:var(--white);border-radius:18px;padding:32px 28px;
    max-width:340px;width:90%;text-align:center;box-shadow:0 12px 40px rgba(100,80,160,.2);}
  .confirm-box p{color:var(--text-dark);font-size:.95rem;margin-bottom:24px;}
  .btn-del{background:#e57373;color:#fff;border:none;border-radius:8px;
    padding:10px 20px;cursor:pointer;font-family:'DM Sans',sans-serif;font-size:.87rem;font-weight:500;}

  /* Toast */
  #toast{position:fixed;bottom:28px;left:50%;transform:translate(-50%,12px);
    background:var(--lav600);color:#fff;border-radius:50px;padding:10px 22px;
    font-size:.85rem;font-weight:500;box-shadow:0 4px 18px rgba(100,80,180,.3);
    z-index:999;white-space:nowrap;opacity:0;
    transition:opacity .3s,transform .3s;pointer-events:none;}

  @media(max-width:640px){
    header,main{padding:18px 16px;}
    .form-grid{grid-template-columns:1fr;}
    .col-notes{display:none;}
  }
</style>
</head>
<body>

<header>
  <div class="brand">
    <div class="brand-icon">🧶</div>
    <div>
      <h1>AK Cozy and Co.</h1>
      <p>Order Tracker</p>
    </div>
  </div>
  <div class="header-right">
    <span id="sync-status">⏳ Connecting…</span>
    <div class="pill"><span class="val" id="stat-year-rev">$0.00</span><span class="lbl">Year Revenue</span></div>
    <div class="pill"><span class="val" id="stat-total-orders">0</span><span class="lbl">Total Orders</span></div>
  </div>
</header>

<main>
  <div class="month-nav">
    <button class="nav-btn" id="btn-prev">&#8592;</button>
    <span id="month-label">Loading…</span>
    <button class="nav-btn today-btn" id="btn-today">Today</button>
    <button class="nav-btn" id="btn-next">&#8594;</button>
  </div>

  <div class="revenue-banner">
    <div>
      <div class="rev-lbl">Monthly Revenue</div>
      <div class="rev-num" id="rev-total">$0.00</div>
    </div>
    <div class="rev-breakdown">
      <div class="rev-stat"><div class="rs-val" id="rev-count">0</div><div class="rs-lbl">Orders</div></div>
      <div class="rev-stat"><div class="rs-val" id="rev-done">0</div><div class="rs-lbl">Completed</div></div>
      <div class="rev-stat"><div class="rs-val" id="rev-prog">0</div><div class="rs-lbl">In Progress</div></div>
      <div class="rev-stat"><div class="rs-val" id="rev-pend">0</div><div class="rs-lbl">Pending</div></div>
    </div>
  </div>

  <div class="toolbar">
    <input id="search-input" placeholder="🔍  Search customer, item, or status…"/>
    <button class="btn-add" id="btn-open-modal">+ Add New Order</button>
  </div>

  <div class="table-wrap">
    <div class="table-scroll">
      <table>
        <thead>
          <tr>
            <th style="cursor:default">#</th>
            <th data-col="customer">Customer ↕</th>
            <th data-col="item">Item ↕</th>
            <th data-col="price">Price ↕</th>
            <th data-col="date">Date ↕</th>
            <th style="cursor:default">Status</th>
            <th class="col-notes" style="cursor:default">Notes</th>
            <th style="cursor:default"></th>
          </tr>
        </thead>
        <tbody id="orders-body"></tbody>
      </table>
      <div id="empty-state">
        <div style="font-size:2.6rem;margin-bottom:12px">🪡</div>
        <p id="empty-msg">No orders for this month yet. Add your first order above!</p>
      </div>
    </div>
  </div>

  <div class="export-row">
    <button class="btn-export" id="btn-export">↓ Export all orders as CSV</button>
  </div>
</main>

<!-- Add / Edit Modal -->
<div class="modal-overlay" id="order-modal">
  <div class="modal">
    <button class="modal-close" id="btn-close-modal">✕</button>
    <h2 id="modal-title">New Order</h2>
    <div class="form-grid">
      <div class="form-row"><label>Customer Name</label><input id="f-customer" placeholder="e.g. Lily Thompson"/></div>
      <div class="form-row"><label>Item</label><input id="f-item" placeholder="e.g. Baby Blanket"/></div>
    </div>
    <div class="form-grid">
      <div class="form-row"><label>Price ($)</label><input type="number" id="f-price" placeholder="0.00" min="0" step="0.01"/></div>
      <div class="form-row"><label>Order Date</label><input type="date" id="f-date"/></div>
    </div>
    <div class="form-row">
      <label>Status</label>
      <select id="f-status">
        <option>Pending</option><option>In Progress</option><option>Done</option><option>Shipped</option>
      </select>
    </div>
    <div class="form-row"><label>Notes</label><textarea id="f-notes" placeholder="Color, size, special requests…"></textarea></div>
    <div id="form-error"></div>
    <div class="modal-footer">
      <button class="btn-cancel" id="btn-cancel-modal">Cancel</button>
      <button class="btn-save" id="btn-save-order">Save Order</button>
    </div>
  </div>
</div>

<!-- Confirm Delete Modal -->
<div class="modal-overlay" id="confirm-modal">
  <div class="confirm-box">
    <div style="font-size:2rem;margin-bottom:12px">🗑️</div>
    <p>Delete this order? This cannot be undone.</p>
    <div style="display:flex;gap:10px;justify-content:center">
      <button class="btn-cancel" id="btn-cancel-delete">Cancel</button>
      <button class="btn-del" id="btn-confirm-delete">Delete</button>
    </div>
  </div>
</div>

<div id="toast"></div>

<script>
// ── Firebase init ─────────────────────────────────────────────
var firebaseConfig = {
  apiKey:            "AIzaSyBDrKn20pa9GKqtVM9pHCg_Amz7sguUAPM",
  authDomain:        "ak-cozy-co.firebaseapp.com",
  databaseURL:       "https://ak-cozy-co-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId:         "ak-cozy-co",
  storageBucket:     "ak-cozy-co.firebasestorage.app",
  messagingSenderId: "784281865983",
  appId:             "1:784281865983:web:ef9c3445d800a594cbc6ca"
};
firebase.initializeApp(firebaseConfig);
var db = firebase.database();
var ordersRef = db.ref("orders");

// ── State ─────────────────────────────────────────────────────
var orders = {};   // keyed by Firebase push key
var editKey = null;
var deleteKey = null;
var sortCol = "date", sortDir = "desc";
var now = new Date();
var curYear = now.getFullYear(), curMonth = now.getMonth();

var MONTHS = ["January","February","March","April","May","June",
              "July","August","September","October","November","December"];
var STATUS_LIST = ["Pending","In Progress","Done","Shipped"];
var STATUS_BG    = {"Pending":"#fef3ec","In Progress":"#eee8fd","Done":"#e9f7ef","Shipped":"#e8f3fd"};
var STATUS_COLOR = {"Pending":"#c47c30","In Progress":"#7a5cc7","Done":"#3a9060","Shipped":"#2d78b8"};

// ── Helpers ───────────────────────────────────────────────────
function fmt(n){ return "$"+Number(n).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g,","); }
function fmtDate(s){ var d=new Date(s+"T00:00:00"); return d.toLocaleDateString("en-US",{month:"short",day:"numeric"}); }
function esc(s){ return String(s||"").replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,"&quot;"); }
function pad(n){ return String(n).padStart(2,"0"); }

function syncStatus(s){
  var el=document.getElementById("sync-status");
  var map={synced:"☁️ Synced",saving:"⏳ Saving…",error:"⚠️ Error",offline:"📴 Offline"};
  var col={synced:"#6b5f8a",saving:"#8469cc",error:"#c06060",offline:"#c47c30"};
  el.textContent=map[s]||"☁️ Synced"; el.style.color=col[s]||"#6b5f8a";
}

function toast(msg){
  var t=document.getElementById("toast");
  t.textContent=msg; t.style.opacity="1"; t.style.transform="translate(-50%,0)";
  setTimeout(function(){ t.style.opacity="0"; t.style.transform="translate(-50%,12px)"; },2400);
}

// ── Real-time listener ────────────────────────────────────────
ordersRef.on("value", function(snap){
  orders = snap.val() || {};
  syncStatus("synced");
  render();
}, function(err){
  console.error(err);
  syncStatus("error");
});

// ── Render ────────────────────────────────────────────────────
function render(){
  var search = document.getElementById("search-input").value.toLowerCase();
  document.getElementById("month-label").textContent = MONTHS[curMonth]+" "+curYear;

  // Convert object to array
  var all = Object.keys(orders).map(function(k){ return Object.assign({},orders[k],{_key:k}); });

  // Filter to current month
  var monthArr = all.filter(function(o){
    var d=new Date(o.date+"T00:00:00");
    return d.getFullYear()===curYear && d.getMonth()===curMonth;
  });

  // Search
  var filtered = monthArr.filter(function(o){
    if(!search) return true;
    return (o.customer||"").toLowerCase().includes(search)||
           (o.item||"").toLowerCase().includes(search)||
           (o.status||"").toLowerCase().includes(search);
  });

  // Sort
  var sorted = filtered.slice().sort(function(a,b){
    var va=a[sortCol], vb=b[sortCol];
    if(sortCol==="price"){ va=Number(va); vb=Number(vb); }
    if(sortCol==="date"){  va=new Date(va); vb=new Date(vb); }
    if(va<vb) return sortDir==="asc"?-1:1;
    if(va>vb) return sortDir==="asc"?1:-1;
    return 0;
  });

  // Stats
  var revenue = monthArr.reduce(function(s,o){ return s+Number(o.price||0); },0);
  var done    = monthArr.filter(function(o){ return o.status==="Done"||o.status==="Shipped"; }).length;
  var prog    = monthArr.filter(function(o){ return o.status==="In Progress"; }).length;
  var pend    = monthArr.filter(function(o){ return o.status==="Pending"; }).length;
  var yearRev = all.filter(function(o){ return new Date(o.date+"T00:00:00").getFullYear()===curYear; })
                   .reduce(function(s,o){ return s+Number(o.price||0); },0);

  document.getElementById("rev-total").textContent  = fmt(revenue);
  document.getElementById("rev-count").textContent  = monthArr.length;
  document.getElementById("rev-done").textContent   = done;
  document.getElementById("rev-prog").textContent   = prog;
  document.getElementById("rev-pend").textContent   = pend;
  document.getElementById("stat-year-rev").textContent    = fmt(yearRev);
  document.getElementById("stat-total-orders").textContent = all.length;

  // Table
  var tbody = document.getElementById("orders-body");
  var empty = document.getElementById("empty-state");

  if(sorted.length===0){
    tbody.innerHTML="";
    empty.style.display="block";
    document.getElementById("empty-msg").textContent =
      search ? "No orders match your search." : "No orders for this month yet. Add your first order above!";
  } else {
    empty.style.display="none";
    tbody.innerHTML = sorted.map(function(o,i){
      var sbg   = STATUS_BG[o.status]   || "#eee";
      var scol  = STATUS_COLOR[o.status] || "#555";
      var opts  = STATUS_LIST.map(function(s){
        return '<option'+(s===o.status?' selected':'')+'>'+s+'</option>';
      }).join("");
      return '<tr>'+
        '<td style="color:#9a90b4;font-size:.76rem">'+(i+1)+'</td>'+
        '<td style="font-weight:500">'+esc(o.customer)+'</td>'+
        '<td>'+esc(o.item)+'</td>'+
        '<td style="font-weight:600">'+fmt(o.price)+'</td>'+
        '<td style="color:#6b5f8a;white-space:nowrap">'+fmtDate(o.date)+'</td>'+
        '<td><select onchange="changeStatus(\''+o._key+'\',this.value)" '+
          'style="border:none;border-radius:50px;padding:3px 10px;font-size:.72rem;font-weight:500;cursor:pointer;'+
          'background:'+sbg+';color:'+scol+'">'+opts+'</select></td>'+
        '<td class="col-notes" style="color:#9a90b4;font-size:.82rem;max-width:160px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap" title="'+esc(o.notes)+'">'+esc(o.notes||"—")+'</td>'+
        '<td style="white-space:nowrap">'+
          '<button class="action-btn" onclick="openEdit(\''+o._key+'\')" title="Edit">✏️</button>'+
          '<button class="action-btn" onclick="askDelete(\''+o._key+'\')" title="Delete">🗑️</button>'+
        '</td>'+
      '</tr>';
    }).join("");
  }
}

// ── Status change ─────────────────────────────────────────────
function changeStatus(key, status){
  syncStatus("saving");
  ordersRef.child(key).update({status:status});
}

// ── Delete ────────────────────────────────────────────────────
function askDelete(key){
  deleteKey=key;
  document.getElementById("confirm-modal").classList.add("open");
}
document.getElementById("btn-cancel-delete").onclick = function(){
  document.getElementById("confirm-modal").classList.remove("open");
  deleteKey=null;
};
document.getElementById("btn-confirm-delete").onclick = function(){
  document.getElementById("confirm-modal").classList.remove("open");
  if(!deleteKey) return;
  syncStatus("saving");
  ordersRef.child(deleteKey).remove();
  deleteKey=null;
  toast("Order deleted");
};

// ── Month nav ─────────────────────────────────────────────────
document.getElementById("btn-prev").onclick = function(){
  if(curMonth===0){curMonth=11;curYear--;}else curMonth--;
  render();
};
document.getElementById("btn-next").onclick = function(){
  if(curMonth===11){curMonth=0;curYear++;}else curMonth++;
  render();
};
document.getElementById("btn-today").onclick = function(){
  var n=new Date(); curYear=n.getFullYear(); curMonth=n.getMonth(); render();
};

// ── Search ────────────────────────────────────────────────────
document.getElementById("search-input").oninput = render;

// ── Sort ──────────────────────────────────────────────────────
document.querySelectorAll("th[data-col]").forEach(function(th){
  th.onclick = function(){
    var col=this.getAttribute("data-col");
    if(sortCol===col) sortDir=sortDir==="asc"?"desc":"asc";
    else{ sortCol=col; sortDir="asc"; }
    render();
  };
});

// ── Modal open/close ──────────────────────────────────────────
function defaultDate(){
  var t=new Date();
  var useDay=(t.getFullYear()===curYear&&t.getMonth()===curMonth)?t.getDate():1;
  return curYear+"-"+pad(curMonth+1)+"-"+pad(useDay);
}

function openAdd(){
  editKey=null;
  document.getElementById("modal-title").textContent="New Order";
  document.getElementById("f-customer").value="";
  document.getElementById("f-item").value="";
  document.getElementById("f-price").value="";
  document.getElementById("f-date").value=defaultDate();
  document.getElementById("f-status").value="Pending";
  document.getElementById("f-notes").value="";
  document.getElementById("form-error").textContent="";
  document.getElementById("order-modal").classList.add("open");
  document.getElementById("f-customer").focus();
}

function openEdit(key){
  var o=orders[key]; if(!o) return;
  editKey=key;
  document.getElementById("modal-title").textContent="Edit Order";
  document.getElementById("f-customer").value=o.customer||"";
  document.getElementById("f-item").value=o.item||"";
  document.getElementById("f-price").value=o.price||"";
  document.getElementById("f-date").value=o.date||defaultDate();
  document.getElementById("f-status").value=o.status||"Pending";
  document.getElementById("f-notes").value=o.notes||"";
  document.getElementById("form-error").textContent="";
  document.getElementById("order-modal").classList.add("open");
  document.getElementById("f-customer").focus();
}

function closeModal(){
  document.getElementById("order-modal").classList.remove("open");
  editKey=null;
}

document.getElementById("btn-open-modal").onclick  = openAdd;
document.getElementById("btn-close-modal").onclick = closeModal;
document.getElementById("btn-cancel-modal").onclick= closeModal;
document.getElementById("order-modal").onclick = function(e){ if(e.target===this) closeModal(); };

// ── Save order ────────────────────────────────────────────────
document.getElementById("btn-save-order").onclick = function(){
  var customer = document.getElementById("f-customer").value.trim();
  var item     = document.getElementById("f-item").value.trim();
  var price    = document.getElementById("f-price").value;
  var date     = document.getElementById("f-date").value;
  var status   = document.getElementById("f-status").value;
  var notes    = document.getElementById("f-notes").value.trim();
  var errEl    = document.getElementById("form-error");

  if(!customer){ errEl.textContent="Customer name is required."; return; }
  if(!item)    { errEl.textContent="Item is required."; return; }
  if(!price||isNaN(Number(price))||Number(price)<0){ errEl.textContent="Enter a valid price."; return; }
  if(!date)    { errEl.textContent="Order date is required."; return; }
  errEl.textContent="";

  var data = { customer:customer, item:item, price:parseFloat(price),
               date:date, status:status, notes:notes };

  syncStatus("saving");
  closeModal();

  if(editKey){
    ordersRef.child(editKey).update(data);
    toast("Order updated ✓");
  } else {
    // Navigate to the month of the new order after save
    var d=new Date(date+"T00:00:00");
    curYear=d.getFullYear(); curMonth=d.getMonth();
    ordersRef.push(data);
    toast("Order added ✓");
  }
};

// ── Export CSV ────────────────────────────────────────────────
document.getElementById("btn-export").onclick = function(){
  var all=Object.values(orders);
  if(!all.length){ toast("No orders to export."); return; }
  var header=["Customer","Item","Price","Date","Status","Notes"];
  var rows=all.map(function(o){
    return [o.customer,o.item,o.price,o.date,o.status,o.notes||""]
      .map(function(v){ return '"'+String(v).replace(/"/g,'""')+'"'; }).join(",");
  });
  var csv=[header.join(",")].concat(rows).join("\n");
  var a=document.createElement("a");
  a.href=URL.createObjectURL(new Blob([csv],{type:"text/csv"}));
  a.download="akcozy-orders.csv"; a.click();
};
</script>
</body>
</html>
