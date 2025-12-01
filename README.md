
<html lang="hi">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>APNA NEWS - Mobile Optimized</title>

<style>
/* GENERAL */
body {
    margin:0;
    font-family:Arial, sans-serif;
    background:#f4f4f4;
}

/* HEADER */
.topbar{
    background:#d60000;
    padding:12px;
    color:#fff;
}
.logo{
    font-size:22px;
    font-weight:bold;
    text-align:center;
}

/* SEARCH + BUTTONS */
.actions{
    margin-top:10px;
    display:flex;
    gap:10px;
    justify-content:center;
}
.actions input{
    flex:1;
    max-width:220px;
    padding:8px;
    border:none;
    border-radius:6px;
}
.actions button{
    background:white;
    border:none;
    border-radius:6px;
    padding:8px 10px;
    font-size:18px;
}

/* NAVBAR */
.navbar{
    background:#fff;
    padding:8px;
    display:flex;
    flex-wrap:wrap;
    justify-content:center;
    gap:12px;
    border-bottom:1px solid #ddd;
}
.navbar a{
    text-decoration:none;
    color:#000;
    font-weight:bold;
    font-size:14px;
}

/* NEWS CARD */
.news-feed{
    padding:12px;
}
.news-card{
    background:#fff;
    border-radius:8px;
    padding:12px;
    margin-bottom:15px;
    box-shadow:0 1px 5px rgba(0,0,0,0.15);
}
.news-card img{
    width:100%;
    border-radius:6px;
    margin-top:8px;
}
.news-card small{
    color:#666;
}

/* AD BLOCK */
.ad-slot{
    margin:12px;
    padding:20px;
    background:#e7e7e7;
    text-align:center;
    border-radius:8px;
    font-size:16px;
}

/* ADMIN MODAL */
.modal-bg{
    position:fixed;
    inset:0;
    background:rgba(0,0,0,0.5);
    display:none;
    align-items:center;
    justify-content:center;
}
.modal{
    background:#fff;
    width:90%;
    max-width:450px;
    padding:15px;
    border-radius:8px;
}
.input{
    padding:8px;
    width:100%;
    border:1px solid #ccc;
    border-radius:5px;
    margin-top:8px;
}
.btn{
    background:#d60000;
    color:white;
    padding:10px;
    border:none;
    width:100%;
    border-radius:5px;
    margin-top:10px;
}
</style>
</head>

<body>

<!-- HEADER -->
<div class="topbar">
    <div class="logo">APNA NEWS</div>

    <div class="actions">
        <input type="text" placeholder="Search...">
        <button onclick="openAdminLogin()">Admin</button>
    </div>
</div>


<!-- NAVIGATION -->
<div class="navbar">
    <a href="#">Home</a>
    <a href="#">Khel</a>
    <a href="#">Chunav</a>
    <a href="#">Daily</a>
    <a href="#">E-Paper</a>
    <a href="#">Reels</a>
    <a href="#">Live</a>
</div>

<!-- AD SECTION -->
<div class="ad-slot">YOUR AD HERE</div>

<!-- NEWS FEED -->
<div class="news-feed" id="news-feed"></div>

<!-- ADMIN LOGIN MODAL -->
<div class="modal-bg" id="loginModal">
    <div class="modal">
        <h2>Admin Login</h2>
        <input id="user" class="input" placeholder="Username">
        <input id="pass" type="password" class="input" placeholder="Password">
        <button class="btn" onclick="login()">Login</button>
        <button class="btn" onclick="closeAdminLogin()" style="background:#555">Close</button>
    </div>
</div>

<script>
const demoNews = [
    {
        title:"India Won The Match!",
        desc:"Bharat ne final match jeet kar itihaas racha...",
        img:"https://i.ibb.co/2hbG6zq/news1.jpg",
        time:"2 min ago"
    },
    {
        title:"Election 2025 Big Update",
        desc:"Naye chunav mein zabardast garma-garmi...",
        img:"https://i.ibb.co/8jHL5jr/news2.jpg",
        time:"12 min ago"
    },
    {
        title:"Weather Alert: Storm Incoming",
        desc:"Aaj shaam tez aandhi aur barish...",
        img:"https://i.ibb.co/5MQtD0J/news3.jpg",
        time:"20 min ago"
    }
];

let feed = document.getElementById("news-feed");

demoNews.forEach(n=>{
    feed.innerHTML += `
    <div class="news-card">
        <h3>${n.title}</h3>
        <img src="${n.img}">
        <p>${n.desc}</p>
        <small>${n.time}</small>
    </div>`;
});

/* ADMIN LOGIN */
function openAdminLogin(){
    document.getElementById("loginModal").style.display="flex";
}
function closeAdminLogin(){
    document.getElementById("loginModal").style.display="none";
}
function login(){
    let u=document.getElementById("user").value;
    let p=document.getElementById("pass").value;
    if(u==="admin" && p==="1234"){
        alert("Login Successful!");
        closeAdminLogin();
    }else{
        alert("Wrong Credentials!");
    }
}
</script>

</body>
</html>
