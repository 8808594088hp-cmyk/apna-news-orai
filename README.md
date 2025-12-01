<!doctype html>
<html lang="hi">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>APNA NEWS — Demo (Single File)</title>
<style>
:root{--brand:#d60000;--muted:#666}
*{box-sizing:border-box}
body{margin:0;font-family:Inter,Arial,Helvetica,sans-serif;background:#f4f6f8;color:#111}
.header{background:var(--brand);color:#fff;padding:10px 14px;display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}
.header .left{display:flex;align-items:center;gap:12px}
.logo{font-weight:800;font-size:20px;letter-spacing:1px}
.top-info{font-size:13px;opacity:.95}
.search-notify{display:flex;align-items:center;gap:8px}
.search-notify input[type="search"]{padding:7px 10px;border-radius:6px;border:none;min-width:160px}
.icon-btn{background:#fff;border-radius:6px;padding:6px 8px;border:none;cursor:pointer;font-size:16px}
.nav{display:flex;gap:12px;padding:10px 14px;background:#fff;border-bottom:1px solid #e2e8f0;flex-wrap:wrap}
.nav a{color:#111;text-decoration:none;font-weight:600;padding:6px 8px;border-radius:6px}
.container{display:flex;gap:18px;padding:16px;align-items:flex-start}
.main{flex:1;min-width:300px}
.sidebar{width:320px;max-width:40%;min-width:260px}
.ad-slot{background:#fff;border:2px dashed #ccc;padding:18px;border-radius:8px;text-align:center;margin-bottom:12px}
.news-card{background:#fff;border-radius:10px;padding:14px;margin-bottom:14px;box-shadow:0 6px 18px rgba(16,24,40,.04)}
.news-card h3{margin:0 0 8px;font-size:18px}
.news-meta{font-size:13px;color:var(--muted);margin-bottom:8px}
.news-card img, .news-card video{width:100%;max-height:360px;object-fit:cover;border-radius:8px;margin:8px 0}
.nav-tabs{display:flex;gap:8px;margin-bottom:12px;flex-wrap:wrap}
.tab{background:#fff;padding:8px 10px;border-radius:8px;cursor:pointer;border:1px solid #e6e9ef;font-weight:600}
.tab.active{background:var(--brand);color:#fff;border-color:var(--brand)}
.controls{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
.small{font-size:13px;color:var(--muted)}
.footer{padding:18px;text-align:center;color:var(--muted);font-size:13px}

/* Admin Modal & Dashboard */
.modal-bg{position:fixed;inset:0;background:rgba(0,0,0,.4);display:none;align-items:center;justify-content:center;z-index:40}
.modal{width:880px;max-width:96%;background:#fff;border-radius:10px;padding:14px;box-shadow:0 20px 60px rgba(2,6,23,.3);max-height:90vh;overflow:auto}
.form-row{display:flex;gap:10px;margin-bottom:10px;flex-wrap:wrap}
.input,textarea,select{padding:8px;border:1px solid #ddd;border-radius:8px;width:100%}
textarea{min-height:90px;resize:vertical}
.btn{background:var(--brand);color:#fff;border:none;padding:9px 12px;border-radius:8px;cursor:pointer;font-weight:700}
.btn.ghost{background:#fff;color:var(--brand);border:1px solid var(--brand)}
.small-muted{font-size:12px;color:#666}
.flex{display:flex;gap:10px;align-items:center}
.kv{font-weight:700;color:#222}
/* responsive */
@media (max-width:900px){
  .container{flex-direction:column;padding:10px}
  .sidebar{width:100%;max-width:100%}
}
</style>
</head>
<body>
# APNA NEWS - Firebase setup (quick)

1. Go to https://console.firebase.google.com
2. Create new Project -> give name "apna-news" or similar.
3. In Project Overview -> Add Web App -> copy firebaseConfig and paste into both files (index.html & admin.html).
4. Enable Authentication -> Email/Password (so admin can create account).
5. Firestore -> Create database (Start in production or test, but set rules as below).
6. Storage -> create default bucket (allow public read for files via rules below).
7. Deploy frontend: push files to GitHub repo and enable GitHub Pages (Settings -> Pages -> branch main -> / (root)).

### Suggested Firestore rules (restrict writes to authenticated users):
<!-- HEADER -->
<header class="header">
  <div class="left">
    <div class="logo">APNA NEWS</div>
    <div class="top-info small">Home • Latest • Live</div>
  </div>

  <div class="search-notify">
    <input type="search" id="searchInput" placeholder="Search news, keywords..." oninput="renderNews()">
    <button class="icon-btn" title="Notifications" onclick="toggleNotifications()">🔔</button>
    <button class="icon-btn" onclick="openAdminLogin()">Admin</button>
  </div>

  <div class="top-info small" id="dt-wthr">—</div>
</header>

<!-- NAV -->
<nav class="nav" id="nav">
  <a href="#" onclick="filterCategory('all');return false">Home</a>
  <a href="#" onclick="filterCategory('khel');return false">Khel</a>
  <a href="#" onclick="filterCategory('chunav');return false">Chunav</a>
  <a href="#" onclick="filterCategory('daily');return false">Daily</a>
  <a href="#" onclick="openEpaper();return false">E-Paper</a>
  <a href="#" onclick="openReels();return false">Reels</a>
  <a href="#" onclick="openLive();return false">Live</a>
</nav>

<!-- LAYOUT -->
<div class="container">
  <main class="main">
    <div class="nav-tabs" id="tabs">
      <div class="tab active" data-cat="all" onclick="selectTab(this)">All</div>
      <div class="tab" data-cat="breaking" onclick="selectTab(this)">Breaking</div>
      <div class="tab" data-cat="khel" onclick="selectTab(this)">Khel</div>
      <div class="tab" data-cat="chunav" onclick="selectTab(this)">Chunav</div>
      <div class="tab" data-cat="daily" onclick="selectTab(this)">Daily</div>
    </div>

    <div class="controls">
      <div class="small-muted">Sort:</div>
      <select id="sortBy" onchange="renderNews()">
        <option value="new">Newest first</option>
        <option value="old">Oldest first</option>
      </select>

      <div style="flex:1"></div>

      <div class="small-muted">Timeline view</div>
    </div>

    <section id="newsList" style="margin-top:12px"></section>

    <div class="footer">APNA NEWS — Demo Portal • Built for preview</div>
  </main>

  <aside class="sidebar">
    <div class="ad-slot">
      <div style="font-weight:800">Ad Slot</div>
      <div class="small-muted">Place your ad code here</div>
    </div>

    <div class="ad-slot" style="padding:10px 12px">
      <div style="font-weight:700">Weather & AQI</div>
      <div id="weatherPanel" class="small-muted" style="margin-top:8px">Loading...</div>
      <div style="margin-top:8px">
        <input id="cityInput" placeholder="Enter city (optional)" style="padding:8px;border-radius:6px;border:1px solid #ddd;width:100%">
        <button class="btn" style="margin-top:8px;width:100%" onclick="fetchWeather()">Get Weather</button>
      </div>
    </div>

    <div class="ad-slot">
      <div style="font-weight:800">Live</div>
      <div style="margin-top:8px">
        <iframe id="liveFrame" width="100%" height="180" src="https://www.youtube.com/embed/5qap5aO4i9A" frameborder="0" allowfullscreen></iframe>
      </div>
    </div>

    <div class="ad-slot">
      <div style="font-weight:700">Quick Actions</div>
      <div style="margin-top:8px;display:flex;gap:8px;flex-direction:column">
        <button class="btn" onclick="openAdminDashboard()">Open Admin Dashboard</button>
        <button class="btn ghost" onclick="exportData()">Export Data</button>
        <button class="btn ghost" onclick="importDataPrompt()">Import Data</button>
      </div>
    </div>

  </aside>
</div>

<!-- ADMIN LOGIN MODAL -->
<div class="modal-bg" id="adminLoginModal">
  <div class="modal" role="dialog" aria-modal="true">
    <h2>Admin Login</h2>
    <div class="form-row">
      <input id="admUser" class="input" placeholder="Username (admin)" value="admin">
      <input id="admPass" class="input" placeholder="Password (1234)" type="password" value="1234">
    </div>
    <div style="display:flex;gap:8px">
      <button class="btn" onclick="doAdminLogin()">Login</button>
      <button class="btn ghost" onclick="closeAdminLogin()">Close</button>
    </div>
    <p class="small-muted" style="margin-top:8px">Demo credentials: admin / 1234</p>
  </div>
</div>
> Hasnain:
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>APNA NEWS — Admin</title>
  <style>
    body{font-family:Inter,Arial,sans-serif;margin:0;padding:18px;background:#fafafa}
    .card{max-width:760px;margin:0 auto;background:#fff;padding:16px;border-radius:8px;box-shadow:0 1px 6px rgba(0,0,0,0.06)}
    label{display:block;margin-top:10px;font-weight:600}
    input,textarea,select{width:100%;padding:8px;margin-top:6px;border-radius:6px;border:1px solid #ddd}
    button{margin-top:12px;padding:10px 14px;border-radius:8px;cursor:pointer;background:#d60000;color:#fff;border:0}
    .row{display:flex;gap:10px}
  </style>
  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-storage-compat.js"></script>
</head>
<body>
  <div class="card">
    <h2>APNA NEWS - Admin Panel</h2>

    <div id="authSection">
      <label>Email</label>
      <input id="email" type="email" placeholder="admin@domain.com">
      <label>Password</label>
      <input id="password" type="password" placeholder="password">
      <div class="row">
        <button id="loginBtn">Login</button>
        <button id="signupBtn" style="background:#666">Create Admin (one-time)</button>
      </div>
      <hr>
    </div>

    <div id="panel" style="display:none">
      <div style="text-align:right"><button id="logoutBtn" style="background:#666">Logout</button></div>
      <label>Title</label><input id="title">
      <label>Category</label><input id="category" placeholder="Politics, Sports, Local...">
      <label>Summary / Content</label><textarea id="summary" rows="4"></textarea>

      <label>Image (jpg/png)</label><input id="imageFile" type="file" accept="image/*">
      <label>Video (mp4) — optional</label><input id="videoFile" type="file" accept="video/*">
      <label>Other file (pdf) — optional</label><input id="otherFile" type="file" accept=".pdf,.doc,.docx">

      <button id="uploadBtn">Upload News</button>
      <div id="status" style="margin-top:8px;color:#333"></div>
    </div>
  </div>

<script>
  // === REPLACE with your firebaseConfig ===
  const firebaseConfig = {
    apiKey: "REPLACE",
    authDomain: "REPLACE",
    projectId: "REPLACE",
    storageBucket: "REPLACE",
    messagingSenderId: "REPLACE",
    appId: "REPLACE"
  };
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();
  const storage = firebase.storage();

  const emailIn = document.getElementById('email');
  const passIn = document.getElementById('password');
  const loginBtn = document.getElementById('loginBtn');
  const signupBtn = document.getElementById('signupBtn');
  const panel = document.getElementById('panel');
  const authSection = document.getElementById('authSection');
  const logoutBtn = document.getElementById('logoutBtn');
  const uploadBtn = document.getElementById('uploadBtn');
  const status = document.getElementById('status');

  auth.onAuthStateChanged(u => {
    if(u){
      authSection.style.display='none';
      panel.style.display='block';
    } else {
      authSection.style.display='block';
      panel.style.display='none';
    }
  });

  signupBtn.onclick = async () => {
    const email = emailIn.value.trim(), pw = passIn.value.trim();
    if(!email || !pw){ alert('Enter email & password'); return; }
    try{
      await auth.createUserWithEmailAndPassword(email,pw);
      alert('Admin account created and logged in.');
    } catch(e){ alert('Error: '+e.message); }
  };

  loginBtn.onclick = async () => {
    try{
      await auth.signInWithEmailAndPassword(emailIn.value.trim(), passIn.value.trim());
    } catch(e){ alert('Login failed: '+e.message); }
  };

  logoutBtn.onclick = () => auth.signOut();

> Hasnain:
uploadBtn.onclick = async () => {
    status.textContent = 'Uploading...';
    const title = document.getElementById('title').value.trim();
    const category = document.getElementById('category').value.trim();
    const summary = document.getElementById('summary').value.trim();
    const imageFile = document.getElementById('imageFile').files[0];
    const videoFile = document.getElementById('videoFile').files[0];
    const otherFile = document.getElementById('otherFile').files[0];

    if(!title){ alert('Title required'); status.textContent=''; return; }

    const docRef = db.collection('news').doc();
    const createdAt = firebase.firestore.FieldValue.serverTimestamp();
    const data = { title, category, summary, createdAt };

    try {
      // Upload image
      if(imageFile){
        const imgRef = storage.ref().child(news/${docRef.id}/image_${Date.now()}_${imageFile.name});
        await imgRef.put(imageFile);
        data.imageURL = await imgRef.getDownloadURL();
      }
      // Upload video
      if(videoFile){
        const vidRef = storage.ref().child(news/${docRef.id}/video_${Date.now()}_${videoFile.name});
        await vidRef.put(videoFile);
        data.videoURL = await vidRef.getDownloadURL();
      }
      // Other file
      if(otherFile){
        const fileRef = storage.ref().child(news/${docRef.id}/file_${Date.now()}_${otherFile.name});
        await fileRef.put(otherFile);
        data.fileURL = await fileRef.getDownloadURL();
      }

      await docRef.set(data);
      status.textContent = 'Uploaded successfully!';
      // clear inputs
      document.getElementById('title').value='';
      document.getElementById('summary').value='';
      document.getElementById('category').value='';
      document.getElementById('imageFile').value='';
      document.getElementById('videoFile').value='';
      document.getElementById('otherFile').value='';
    } catch(err){
      console.error(err);
      alert('Upload error: '+err.message);
      status.textContent = 'Error during upload';
    }
  };
</script>
</body>
</html>

<!-- ADMIN DASHBOARD (Add/Edit/Delete) -->
<div class="modal-bg" id="adminDashModal">
  <div class="modal" role="dialog">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <h2>Admin Dashboard — Manage News</h2>
      <div style="display:flex;gap:8px">
        <button class="btn ghost" onclick="closeAdminDashboard()">Close</button>
        <button class="btn" onclick="adminLogout()">Logout</button>
      </div>
    </div>

    <hr>

    <h3>Add / Edit News</h3>
    <div class="form-row">
      <input id="newsTitle" class="input" placeholder="Title">
      <select id="newsCategory" class="input" style="max-width:200px">
        <option value="breaking">Breaking</option>
        <option value="khel">Khel</option>
        <option value="chunav">Chunav</option>
        <option value="daily">Daily</option>
        <option value="other">Other</option>
      </select>
    </div>
    <div class="form-row">
      <textarea id="newsDesc" placeholder="Short description..." class="input"></textarea>
    </div>

    <div class="form-row">
      <label class="small-muted">Image / Video (optional)</label>
      <input type="file" id="mediaFile" accept="image/*,video/*">
    </div>

    <div class="form-row">
      <input id="newsSource" class="input" placeholder="Source / Author (optional)">
      <input id="newsTime" class="input" placeholder="Display time (auto if empty)">
    </div>

    <div style="display:flex;gap:8px;margin-bottom:8px">
      <button class="btn" onclick="saveNews()">Save News</button>
      <button class="btn ghost" onclick="clearNewsForm()">Clear</button>
      <div style="flex:1"></div>
      <div class="small-muted kv">Total: <span id="totalNews">0</span></div>
    </div>

    <hr>
    <h3>Existing News</h3>
    <div id="adminNewsList"></div>

  </div>
</div>

<!-- E-PAPER MODAL -->
<div class="modal-bg" id="epaperModal">
  <div class="modal">
    <h2>E-Paper / Daily</h2>
    <p class="small-muted">यह एक demo E-paper modal है. यहाँ आप PDF embed कर सकते हैं या e-paper images दिखा सकते हैं.</p>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:12px">
      <div style="background:#fff;padding:12px;border-radius:8px;min-width:200px">
        <strong>EPAPER - 30 Nov 2025</strong>
        <p class="small-muted">Sample front page</p>
        <img src="https://via.placeholder.com/300x200?text=E-Paper" style="width:100%;border-radius:6px">
      </div>
      <div style="flex:1"></div>
    </div>
    <div style="margin-top:12px"><button class="btn" onclick="closeEpaper()">Close</button></div>
  </div>
</div>

<!-- REELS MODAL -->
<div class="modal-bg" id="reelsModal">
  <div class="modal">
    <h2>Reels / Short Videos</h2>
    <p class="small-muted">Short video grid (uploaded / demo)</p>
    <div id="reelsGrid" style="display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:8px;margin-top:12px"></div>
    <div style="margin-top:12px"><button class="btn" onclick="closeReels()">Close</button></div>
  </div>
</div>

<!-- Notifications panel (simple) -->
<div style="position:fixed;right:12px;bottom:12px;z-index:60" id="notifyArea"></div>

<script>
/* -------------------------
   Storage & Initial Setup
   -------------------------*/
const STORAGE_KEY = 'apnaNewsData_v1';
const ADMIN_KEY = 'apnaNewsAdmin_v1';
let appState = { filter:'all', search:'', sort:'new', loggedIn:false, editingId:null };

// default demo content if none
if(!localStorage.getItem(STORAGE_KEY)){
  const demo = [
    { id: genId(), title:"India Won The Match!", desc:"Bharat ne final match jeet kar itihaas racha...", category:"khel", src:"APNA", time:Date.now()-2*60*1000, mediaType:'image', mediaData:'https://i.ibb.co/2hbG6zq/news1.jpg' },
    { id: genId(), title:"Election 2025 Big Update", desc:"Naye chunav mein kaafi garma-garmi dekhne ko...", category:"chunav", src:"Reporter", time:Date.now()-12*60*1000, mediaType:'image', mediaData:'https://i.ibb.co/8jHL5jr/news2.jpg' },
    { id: genId(), title:"Weather Alert: Storm Incoming", desc:"Aaj sham tak tez aandhi aur barish ki sambhavna...", category:"breaking", src:"Weather Desk", time:Date.now()-20*60*1000, mediaType:'image', mediaData:'https://i.ibb.co/5MQtD0J/news3.jpg' }
  ];
  localStorage.setItem(STORAGE_KEY, JSON.stringify(demo));
}
if(!localStorage.getItem(ADMIN_KEY)){
  localStorage.setItem(ADMIN_KEY, JSON.stringify({user:'admin',pass:'1234'}));
}

/* -------------------------
   Utility functions
   -------------------------*/
function genId(){ return 'n_'+Math.random().toString(36).slice(2,9) }

function readData(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY)||'[]') }catch(e){return []} }
function writeData(d){ localStorage.setItem(STORAGE_KEY, JSON.stringify(d)); }

function timeAgo(ts){
  const sec=Math.floor((Date.now()-ts)/1000);
  if(sec<60) return sec+' sec ago';
  if(sec<3600) return Math.floor(sec/60)+' min ago';
  if(sec<86400) return Math.floor(sec/3600)+' hr ago';
  return new Date(ts).toLocaleString();
}

/* -------------------------
   Render news list (main)
   -------------------------*/
function renderNews(){
  const container = document.getElementById('newsList');
  const search = document.getElementById('searchInput').value.trim().toLowerCase();
  appState.search = search;
  appState.sort = document.getElementById('sortBy').value;
  let data = readData();

  // filters
  if(appState.filter && appState.filter!=='all'){
    data = data.filter(n => n.category === appState.filter);
  }
  if(search){
    data = data.filter(n => (n.title+' '+n.desc+' '+(n.src||'')).toLowerCase().includes(search));
  }
  // sort
  data.sort((a,b)=> appState.sort==='new' ? b.time - a.time : a.time - b.time);

  // timeline style: simply ordered cards
  container.innerHTML = data.map(n=>{
    const mediaHtml = n.mediaType === 'image'
      ? `<img src="${escapeHtml(n.mediaData)}" alt="">`
      : n.mediaType === 'video'
        ? `<video controls src="${escapeHtml(n.mediaData)}"></video>`
        : '';
    return `<article class="news-card" id="${n.id}">
      <div class="news-meta">${escapeHtml(n.category.toUpperCase())} • <span class="small-muted">${timeAgo(n.time)}</span></div>
      <h3>${escapeHtml(n.title)}</h3>
      ${mediaHtml}
      <p>${escapeHtml(n.desc)}</p>
      <div class="small-muted">Source: ${escapeHtml(n.src||'APNA')}</div>
      ${appState.loggedIn ? `<div style="margin-top:8px;display:flex;gap:8px">
         <button class="btn ghost" onclick="startEdit('${n.id}')">Edit</button>
         <button class="btn" onclick="deleteNews('${n.id}')">Delete</button>
      </div>` : ''}
    </article>`;
  }).join('') || '<div class="small-muted">Koi bhi news nahi mili — add karo admin se.</div>';

  document.getElementById('totalNews').innerText = data.length;
}

/* -------------------------
   Admin: Save / Edit / Delete
   -------------------------*/
function openAdminLogin(){ document.getElementById('adminLoginModal').style.display='flex' }
function closeAdminLogin(){ document.getElementById('adminLoginModal').style.display='none' }

function doAdminLogin(){
  const user = document.getElementById('admUser').value;
  const pass = document.getElementById('admPass').value;
  const admin = JSON.parse(localStorage.getItem(ADMIN_KEY));
  if(user===admin.user && pass===admin.pass){
    appState.loggedIn = true;
    showToast('Logged in as Admin', 'success');
    closeAdminLogin();
    openAdminDashboard();
    renderNews();
  } else {
    showToast('Invalid credentials', 'error');
  }
}

function adminLogout(){
  appState.loggedIn = false;
  closeAdminDashboard();
  renderNews();
  showToast('Logged out', 'info');
}

function openAdminDashboard(){
  if(!appState.loggedIn){ openAdminLogin(); return; }
  document.getElementById('adminDashModal').style.display='flex';
  refreshAdminList();
}

function closeAdminDashboard(){ document.getElementById('adminDashModal').style.display='none'; clearNewsForm(); }

function saveNews(){
  const title = document.getElementById('newsTitle').value.trim();
  const desc = document.getElementById('newsDesc').value.trim();
  const cat = document.getElementById('newsCategory').value;
  const src = document.getElementById('newsSource').value.trim();
  const timeField = document.getElementById('newsTime').value.trim();

  if(!title){ showToast('Title चाहिए', 'error'); return; }

  const file = document.getElementById('mediaFile').files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = () => {
      const base = reader.result;
      persistNews(title, desc, cat, src, timeField, base, file.type.startsWith('video')?'video':'image');
    };
    reader.readAsDataURL(file);
  } else {
    persistNews(title, desc, cat, src, timeField, null, null);
  }
}

function persistNews(title, desc, cat, src, timeField, mediaData, mediaType){
  const data = readData();
  if(appState.editingId){
    // update
    const idx = data.findIndex(d=>d.id===appState.editingId);
    if(idx===-1) return;
    data[idx].title = title;
    data[idx].desc = desc;
    data[idx].category = cat;
    data[idx].src = src;
    data[idx].time = timeField ? new Date(timeField).getTime() : Date.now();
    if(mediaData){ data[idx].mediaData = mediaData; data[idx].mediaType = mediaType; }
    writeData(data);
    showToast('News updated', 'success');
    appState.editingId = null;
  } else {
    const obj = { id: genId(), title, desc, category:cat, src, time: timeField ? new Date(timeField).getTime() : Date.now(), mediaType: mediaType||null, mediaData: mediaData||null };
    data.push(obj);
    writeData(data);
    showToast('News added', 'success');
  }
  clearNewsForm();
  refreshAdminList();
  renderNews();
}

function clearNewsForm(){
  document.getElementById('newsTitle').value='';
  document.getElementById('newsDesc').value='';
  document.getElementById('mediaFile').value='';
  document.getElementById('newsSource').value='';
  document.getElementById('newsTime').value='';
  appState.editingId = null;
}

function refreshAdminList(){
  const list = readData().slice().sort((a,b)=>b.time-a.time);
  const el = document.getElementById('adminNewsList');
  el.innerHTML = list.map(n=>`<div style="background:#fff;padding:8px;border-radius:8px;margin-bottom:8px">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div><strong>${escapeHtml(n.title)}</strong><div class="small-muted">${escapeHtml(n.category)} • ${timeAgo(n.time)}</div></div>
      <div style="display:flex;gap:6px">
        <button class="btn ghost" onclick="startEdit('${n.id}')">Edit</button>
        <button class="btn" onclick="deleteNews('${n.id}')">Delete</button>
      </div>
    </div>
  </div>`).join('');
}

function startEdit(id){
  const data = readData();
  const item = data.find(d=>d.id===id);
  if(!item) return;
  appState.editingId = id;
  openAdminDashboard();
  document.getElementById('newsTitle').value = item.title;
  document.getElementById('newsDesc').value = item.desc;
  document.getElementById('newsCategory').value = item.category;
  document.getElementById('newsSource').value = item.src || '';
  document.getElementById('newsTime').value = new Date(item.time).toISOString().slice(0,16);
  showToast('Edit mode – media replace optional', 'info');
}

function deleteNews(id){
  if(!confirm('Are you sure to delete this news?')) return;
  let data = readData();
  data = data.filter(d=>d.id!==id);
  writeData(data);
  refreshAdminList();
  renderNews();
  showToast('Deleted', 'info');
}

/* -------------------------
   Filters / Tabs
   -------------------------*/
function selectTab(el){
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  el.classList.add('active');
  filterCategory(el.dataset.cat);
}
function filterCategory(cat){
  appState.filter = cat;
  renderNews();
}

/* -------------------------
   E-Paper / Reels / Live
   -------------------------*/
function openEpaper(){ document.getElementById('epaperModal').style.display='flex' }
function closeEpaper(){ document.getElementById('epaperModal').style.display='none' }

function openReels(){
  document.getElementById('reelsModal').style.display='flex';
  renderReels();
}
function closeReels(){ document.getElementById('reelsModal').style.display='none' }

function renderReels(){
  const grid = document.getElementById('reelsGrid');
  const reels = readData().filter(n=>n.mediaType==='video');
  if(reels.length===0){
    grid.innerHTML = '<div class="small-muted">No reels uploaded yet.</div>';
    return;
  }
  grid.innerHTML = reels.map(r=>{
    return `<div style="background:#fff;border-radius:8px;overflow:hidden">
      <video controls style="width:100%;height:200px;object-fit:cover" src="${escapeHtml(r.mediaData)}"></video>
      <div style="padding:8px"><strong>${escapeHtml(r.title)}</strong><div class="small-muted">${escapeHtml(r.category)}</div></div>
    </div>`;
  }).join('');
}

function openLive(){
  window.scrollTo({top:0,behavior:'smooth'});
  showToast('Live opened in side panel', 'info');
  // focus the iframe (already present)
}

/* -------------------------
   Weather (simple demo)
   -------------------------*/
async function fetchWeather(){
  const city = document.getElementById('cityInput').value.trim();
  const panel = document.getElementById('weatherPanel');
  panel.innerHTML = 'Fetching...';
  try{
    // Try to use open-meteo (no API key) by geolocation or city -> naive approach
    if(city){
      // Use geocoding via open-meteo (free)
      const g = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1`).then(r=>r.json());
      if(!g.results || g.results.length===0){ panel.innerText='City not found'; return; }
      const loc = g.results[0];
      const weather = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${loc.latitude}&longitude=${loc.longitude}&current_weather=true`).then(r=>r.json());
      panel.innerHTML = `${loc.name}, ${loc.country}<br>Temp: ${weather.current_weather.temperature}°C • Wind: ${weather.current_weather.windspeed} km/h`;
    } else if(navigator.geolocation){
      navigator.geolocation.getCurrentPosition(async pos=>{
        const lat = pos.coords.latitude; const lon = pos.coords.longitude;
        const weather = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`).then(r=>r.json());
        panel.innerHTML = `Nearby • Temp: ${weather.current_weather.temperature}°C • Wind: ${weather.current_weather.windspeed} km/h`;
      }, err=>{
        panel.innerHTML = 'Location denied — enter orai city';
      }, {timeout:10000});
    } else {
      panel.innerHTML = 'Geolocation not supported';
    }
  }catch(e){
    panel.innerHTML = 'Weather fetch failed (no internet?)';
  }
}

/* -------------------------
   Notifications & Toasts
   -------------------------*/
function toggleNotifications(){
  showToast('Sample Notification: Breaking news!', 'info');
  // also try system notification
  if("Notification" in window){
    if(Notification.permission==="granted"){
      new Notification("APNA NEWS", {body:"This is a sample notification from Apna News."});
    } else if(Notification.permission!=="denied"){
      Notification.requestPermission(permission=>{
        if(permission==="granted") new Notification("APNA NEWS", {body:"Notifications enabled."});
      });
    }
  }
}
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>APNA NEWS - Live</title>
  <style>
    body{font-family:Inter,Arial,sans-serif; background:#f7f7f8; margin:0; padding:16px; color:#111}
    .container{max-width:900px;margin:0 auto;}
    header{display:flex;align-items:center;gap:12px;margin-bottom:18px}
    header h1{margin:0;font-size:24px;color:#d60000}
    .card{background:#fff;border-radius:8px;padding:12px;margin-bottom:12px;box-shadow:0 1px 4px rgba(0,0,0,0.06)}
    .meta{font-size:12px;color:#666;margin-bottom:8px}
    img.resp{max-width:100%;border-radius:6px;margin-top:8px}
    video{max-width:100%;margin-top:8px;border-radius:6px}
  </style>
  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-storage-compat.js"></script>
</head>
<body>
  <div class="container">
    <header>
      <h1>APNA NEWS</h1>
      <div style="margin-left:auto"><a href="admin.html">Admin Login</a></div>
    </header>

    <div id="feed">Loading news...</div>
  </div>

<script>
  // === REPLACE with your firebaseConfig from Firebase console ===
  const firebaseConfig = {
    apiKey: "REPLACE",
    authDomain: "REPLACE",
    projectId: "REPLACE",
    storageBucket: "REPLACE",
    messagingSenderId: "REPLACE",
    appId: "REPLACE"
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const storage = firebase.storage();

  // Fetch latest news (ordered by createdAt desc)
  async function loadNews(){
    const feed = document.getElementById('feed');
    feed.innerHTML = 'Loading news...';
    try {
      const snap = await db.collection('news').orderBy('createdAt','desc').limit(50).get();
      if(snap.empty){ feed.innerHTML = '<p>No news yet.</p>'; return; }
      let html = '';
      snap.forEach(doc => {
        const d = doc.data();
        const time = d.createdAt && d.createdAt.toDate ? d.createdAt.toDate().toLocaleString('hi-IN') : '';
        html += <div class="card">
                  <div class="meta">${d.category || 'General'} • ${time}</div>
                  <h3>${escapeHtml(d.title || '')}</h3>
                  <p>${escapeHtml(d.summary || '')}</p>;
        if(d.imageURL){
          html += <img class="resp" src="${d.imageURL}" alt="news image">;
        }
        if(d.videoURL){
          html += <video controls src="${d.videoURL}"></video>;
        }
        if(d.fileURL){
          html += <div style="margin-top:8px"><a target="_blank" href="${d.fileURL}">Download attachment</a></div>;
        }
        html += </div>;
      });
      feed.innerHTML = html;
    } catch(err){
      console.error(err);
      feed.innerHTML = '<p>Error loading news.</p>';
    }
  }

  function escapeHtml(s){
    if(!s) return '';
    return s.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  }

  loadNews();

  // Optional: realtime updates (uncomment to enable)
  db.collection('news').orderBy('createdAt','desc').limit(50)
    .onSnapshot(snapshot => { loadNews(); });
</script>
</body>
</html>
function showToast(msg, type='info', timeout=2500){
  const area = document.getElementById('notifyArea');
  const el = document.createElement('div');
  el.style.background = type==='error' ? '#ffdddd' : type==='success' ? '#e6ffed' : '#fff';
  el.style.border = '1px solid #ddd'; el.style.padding='10px 12px'; el.style.borderRadius='8px';
  el.style.boxShadow='0 8px 24px rgba(2,6,23,.08)'; el.style.marginTop='8px';
  el.innerText = msg;
  area.appendChild(el);
  setTimeout(()=>{ el.style.opacity=0; setTimeout(()=>el.remove(),400); }, timeout);
}

/* -------------------------
   Export / Import data
   -------------------------*/
function exportData(){
  const dataStr = localStorage.getItem(STORAGE_KEY);
  const blob = new Blob([dataStr], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href=url; a.download='apna_news_data.json'; a.click(); URL.revokeObjectURL(url);
}

function importDataPrompt(){
  const inp = document.createElement('input'); inp.type='file'; inp.accept='.json';
  inp.onchange = e=>{
    const f = e.target.files[0]; if(!f) return;
    const reader = new FileReader();
    reader.onload = ()=>{ try{ JSON.parse(reader.result); localStorage.setItem(STORAGE_KEY, reader.result); showToast('Imported', 'success'); renderNews(); }catch(err){ showToast('Invalid file', 'error'); } };
    reader.readAsText(f);
  };
  inp.click();
}

/* -------------------------
   Helpers: escapeHtml
   -------------------------*/
function escapeHtml(s){
  if(!s && s!==0) return '';
  return String(s).replace(/[&<>"'`]/g, function(m){ return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','`':'&#96;'}[m]; });
}

/* -------------------------
   Init: date/time + first render
   -------------------------*/
function tickDateTime(){
  const el = document.getElementById('dt-wthr');
  const d = new Date();
  el.innerText = d.toLocaleDateString('hi-IN',{day:'numeric',month:'short',year:'numeric'}) + ' • ' + d.toLocaleTimeString();
}
setInterval(tickDateTime,1000); tickDateTime();

renderNews();

/* -------------------------
   Small helpers for external actions
   -------------------------*/
function startDemoAdd(){
  openAdminLogin();
}
function toggleAdmin(){
  if(appState.loggedIn) openAdminDashboard(); else openAdminLogin();
}

/* -------------------------
   Simple escape for security and finishing touches
   -------------------------*/
document.addEventListener('keydown', ev=>{
  if(ev.key==='Escape'){
    document.querySelectorAll('.modal-bg').forEach(m=>m.style.display='none');
  }
});

// close modals clicking outside
document.querySelectorAll('.modal-bg').forEach(bg=>{
  bg.addEventListener('click', e=>{
    if(e.target===bg) bg.style.display='none';
  });
});
</script>

</body>
</html>
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>APNA NEWS - Live</title>
  <style>
    body{font-family:Inter,Arial,sans-serif; background:#f7f7f8; margin:0; padding:16px; color:#111}
    .container{max-width:900px;margin:0 auto;}
    header{display:flex;align-items:center;gap:12px;margin-bottom:18px}
    header h1{margin:0;font-size:24px;color:#d60000}
    .card{background:#fff;border-radius:8px;padding:12px;margin-bottom:12px;box-shadow:0 1px 4px rgba(0,0,0,0.06)}
    .meta{font-size:12px;color:#666;margin-bottom:8px}
    img.resp{max-width:100%;border-radius:6px;margin-top:8px}
    video{max-width:100%;margin-top:8px;border-radius:6px}
  </style>
  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.24.0/firebase-storage-compat.js"></script>
</head>
<body>
  <div class="container">
    <header>
      <h1>APNA NEWS</h1>
      <div style="margin-left:auto"><a href="admin.html">Admin Login</a></div>
    </header>

    <div id="feed">Loading news...</div>
  </div>

<script>
  // === REPLACE with your firebaseConfig from Firebase console ===
  const firebaseConfig = {
    apiKey: "REPLACE",
    authDomain: "REPLACE",
    projectId: "REPLACE",
    storageBucket: "REPLACE",
    messagingSenderId: "REPLACE",
    appId: "REPLACE"
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const storage = firebase.storage();

  // Fetch latest news (ordered by createdAt desc)
  async function loadNews(){
    const feed = document.getElementById('feed');
    feed.innerHTML = 'Loading news...';
    try {
      const snap = await db.collection('news').orderBy('createdAt','desc').limit(50).get();
      if(snap.empty){ feed.innerHTML = '<p>No news yet.</p>'; return; }
      let html = '';
      snap.forEach(doc => {
        const d = doc.data();
        const time = d.createdAt && d.createdAt.toDate ? d.createdAt.toDate().toLocaleString('hi-IN') : '';
        html += <div class="card">
                  <div class="meta">${d.category || 'General'} • ${time}</div>
                  <h3>${escapeHtml(d.title || '')}</h3>
                  <p>${escapeHtml(d.summary || '')}</p>;
        if(d.imageURL){
          html += <img class="resp" src="${d.imageURL}" alt="news image">;
        }
        if(d.videoURL){
          html += <video controls src="${d.videoURL}"></video>;
        }
        if(d.fileURL){
          html += <div style="margin-top:8px"><a target="_blank" href="${d.fileURL}">Download attachment</a></div>;
        }
        html += </div>;
      });
      feed.innerHTML = html;
    } catch(err){
      console.error(err);
      feed.innerHTML = '<p>Error loading news.</p>';
    }
  }

  function escapeHtml(s){
    if(!s) return '';
    return s.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  }

  loadNews();

  // Optional: realtime updates (uncomment to enable)
  db.collection('news').orderBy('createdAt','desc').limit(50)
    .onSnapshot(snapshot => { loadNews(); });
</script>
</body>
</html>
