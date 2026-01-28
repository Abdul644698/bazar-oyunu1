<!DOCTYPE html>
<html lang="az">
<head>
<meta charset="UTF-8">
<title>Bazar Oyunu – Vergi Sistemi</title>
<style>
body{margin:0;font-family:Arial;background:#0b132b;color:white}
#top{background:#000a;padding:10px;display:flex;justify-content:space-between}
.box{background:#1c2541;margin:15px;padding:15px;border-radius:10px}
button,input,select{padding:7px;margin:4px;border-radius:5px;border:none}
button{background:#5bc0be;color:black;cursor:pointer}
.item{background:#3a506b;padding:8px;margin:6px 0;border-radius:5px;display:flex;justify-content:space-between;align-items:center}
.admin{background:#fca311;color:black}
input.priceInput{width:60px;margin-left:10px}
</style>
</head>
<body>

<div id="top">
<span id="userInfo">Giriş yoxdur</span>
<span>Pul: <b><span id="userMoney">0</span> AZN</b></span>
</div>

<!-- LOGIN -->
<div class="box" id="loginBox">
<h2>Giriş / Qeydiyyat</h2>
<input id="username" placeholder="İstifadəçi adı"><br>
<input id="password" type="password" placeholder="Şifrə"><br>
<button onclick="login()">Daxil ol</button>
</div>

<!-- MENU -->
<div class="box" id="menu" style="display:none">
<button onclick="show('wholesale')">🏭 Toptancı</button>
<button onclick="show('myMarket')">🏪 Öz Bazarım</button>
<button onclick="show('globalMarket')">🌍 Bazardakı mallar</button>
<button onclick="show('sendMoneyBox')">💸 Pul Göndər</button>
<button onclick="show('adminPanel')">👑 Admin Panel</button>
<button onclick="logout()">Çıxış</button>
</div>

<!-- TOPTANCI -->
<div class="box" id="wholesale" style="display:none">
<h3>🏭 Toptancı (Pullu)</h3>
<div id="wholesaleItems"></div>
</div>

<!-- Öz bazar -->
<div class="box" id="myMarket" style="display:none">
<h3>Öz Bazarın</h3>
<div id="myItems"></div>
</div>

<!-- Bazardakı mallar -->
<div class="box" id="globalMarket" style="display:none">
<h3>🌍 Bazardakı Mallar</h3>
<div id="globalItems"></div>
</div>

<!-- Pul göndər -->
<div class="box" id="sendMoneyBox" style="display:none">
<h3>💸 Pul Göndər</h3>
<select id="sendUser"></select>
<input type="number" id="sendAmount" placeholder="Məbləğ">
<button onclick="sendMoney()">Göndər</button>
</div>

<!-- ADMIN PANEL -->
<div class="box admin" id="adminPanel" style="display:none">
<h3>👑 Admin Panel</h3>
<p>Admin pulu: <b id="adminMoneyDisplay"></b> AZN</p>

<h4>İstifadəçilər</h4>
<div id="adminUsers"></div>

<h4>Topdan mallar</h4>
<input id="wName" placeholder="Topdan mal adı">
<input id="wPrice" type="number" placeholder="Topdan qiymət AZN">
<button onclick="addWholesale()">Əlavə et</button>
<div id="adminWholesale"></div>
</div>

<script>
// --- DATA ---
let users = JSON.parse(localStorage.getItem("users")) || {};          
let wholesale = JSON.parse(localStorage.getItem("wholesale")) || [
  {name:"Çörək", price:2}, {name:"Süd", price:3}
];      
let market = JSON.parse(localStorage.getItem("market")) || [];         
let admin = localStorage.getItem("admin") || null;        
let currentUser = localStorage.getItem("currentUser") || null;  
let adminMoney = Number(localStorage.getItem("adminMoney")) || 999999999; // adminin böyük pulu

// --- SAVE ---
function save(){
 localStorage.setItem("users",JSON.stringify(users));
 localStorage.setItem("wholesale",JSON.stringify(wholesale));
 localStorage.setItem("market",JSON.stringify(market));
 localStorage.setItem("admin",admin);
 localStorage.setItem("currentUser",currentUser);
 localStorage.setItem("adminMoney",adminMoney);
}

// --- LOGOUT ---
function logout(){
 currentUser=null;
 localStorage.removeItem("currentUser");
 location.reload();
}

// --- LOGIN ---
function login(){
 let u=username.value.trim();
 let p=password.value;
 if(!u||!p){ alert("Boş olmaz"); return; }

 if(!admin){
   admin=u;
   users[u]={password:p,money:adminMoney,stock:[]};
   alert("Siz ADMIN oldunuz, pul: " + adminMoney + " AZN");
 } else if(u===admin){
   if(users[u].password!==p){alert("Şifrə səhvdir"); return;}
 } else if(!users[u]){
   users[u]={password:p,money:10,stock:[]};
   alert("Yeni istifadəçi yaradıldı, pul: 10 AZN");
 } else if(users[u].password!==p){alert("Şifrə səhvdir"); return;}
 
 currentUser=u;
 save();
 start();
}

// --- START ---
function start(){
 loginBox.style.display="none";
 menu.style.display="block";
 userInfo.innerText=currentUser+(currentUser===admin?" (ADMIN)":"");

updateUserMoney();
loadWholesale();
loadMy();
loadMarket();
loadSendUsers();
loadAdminPanel();
show('wholesale');
}

// --- SHOW ---
function show(id){
 document.querySelectorAll(".box").forEach(b=>b.style.display="none");
 menu.style.display="block";
 document.getElementById(id).style.display="block";
}

// --- UPDATE MONEY ---
function updateUserMoney(){
 let money = currentUser===admin ? adminMoney : users[currentUser].money;
 document.getElementById("userMoney").innerText = money;
}

// --- LOAD TOPTANCI ---
function loadWholesale(){
 wholesaleItems.innerHTML="";
 wholesale.forEach((i,idx)=>{
   let btnDel = currentUser===admin ? `<button onclick="deleteWholesale(${idx})">Sil</button>` : '';
   let btnBuy = currentUser===admin ? '' : `<button onclick="buyWholesale(${idx})">Al</button>`;
   wholesaleItems.innerHTML+=`<div class="item">${i.name} | Qiymət: ${i.price} AZN ${btnBuy}${btnDel}</div>`;
 });
}

// --- DELETE TOPTANCI ---
function deleteWholesale(idx){
 wholesale.splice(idx,1);
 save();
 loadWholesale();
 loadAdminPanel();
}

// --- BUY TOPTANCI ---
function buyWholesale(i){
 let price = wholesale[i].price;
 if(users[currentUser].money < price){ alert("Pul çatmır"); return; }
 users[currentUser].money -= price;
 users[currentUser].stock.push({...wholesale[i]});
 // 5% admin vergisi topdan alışda
 let tax = price * 0.05;
 adminMoney += tax;
 save();
 loadMy();
 loadWholesale();
 updateUserMoney();
}

// --- LOAD MY STOCK ---
function loadMy(){
 myItems.innerHTML="";
 if(currentUser!==admin){
 users[currentUser].stock.forEach((i,idx)=>{
   myItems.innerHTML+=`<div class="item">
     ${i.name} | Əvvəlki qiymət: ${i.price} AZN
     <input class="priceInput" type="number" placeholder="Satış AZN" id="sellPrice${idx}">
     <button onclick="sellItem(${idx})">Sat</button>
   </div>`;
 });
 }
}

// --- SELL ITEM ---
function sellItem(idx){
 let priceInput = document.getElementById("sellPrice"+idx);
 let price = Number(priceInput.value);
 if(price<=0){alert("Qiyməti düzgün daxil et"); return;}
 let item = users[currentUser].stock.splice(idx,1)[0];
 // Vergi 5%
 let tax = price * 0.05;
 let sellerGets = price - tax;
 market.push({...item,price:sellerGets,seller:currentUser});
 adminMoney += tax;
 save();
 loadMy();
 loadMarket();
 updateUserMoney();
}

// --- LOAD MARKET ---
function loadMarket(){
 globalItems.innerHTML="";
 market.forEach((i,idx)=>{
   let btnDel = currentUser===admin ? `<button onclick="deleteMarket(${idx})">Sil</button>` : '';
   let btnBuy = currentUser===admin ? `<button onclick="buyFromMarket(${idx})">Al</button>` : `<button onclick="buyFromMarket(${idx})">Al</button>`;
   globalItems.innerHTML+=`<div class="item">${i.name} | Qiymət: ${i.price} AZN | Satıcı: ${i.seller} ${btnBuy}${btnDel}</div>`;
 });
}

// --- DELETE MARKET ITEM ---
function deleteMarket(idx){
 market.splice(idx,1);
 save();
 loadMarket();
}

// --- BUY FROM MARKET ---
function buyFromMarket(idx){
 let item = market[idx];
 if(users[currentUser].money < item.price){ alert("Pul çatmır"); return; }
 users[currentUser].money -= item.price;
 users[item.seller].money += item.price;
 // Vergi 5%
 let tax = item.price * 0.05;
 users[item.seller].money -= tax;
 adminMoney += tax;
 users[currentUser].stock.push({...item});
 market.splice(idx,1);
 save();
 loadMy();
 loadMarket();
 updateUserMoney();
}

// --- PUL GÖNDƏR ---
function loadSendUsers(){
 sendUser.innerHTML="";
 Object.keys(users).forEach(u=>{
   if(u!==currentUser) sendUser.innerHTML+=`<option value="${u}">${u}</option>`;
 });
}

function sendMoney(){
 let user = sendUser.value;
 let amount = Number(sendAmount.value);
 if(!user || amount<=0){ alert("Düzgün daxil et"); return; }
 if(currentUser===admin){
   if(adminMoney<amount){ alert("Pul çatmır"); return; }
   adminMoney-=amount;
   users[user].money+=amount;
 } else {
   if(users[currentUser].money<amount){ alert("Pul çatmır"); return; }
   users[currentUser].money-=amount;
   users[user].money+=amount;
 }
 save();
 updateUserMoney();
 alert(amount + " AZN göndərildi "+user+"-ə");
 loadSendUsers();
}

// --- ADMIN PANEL ---
function addWholesale(){
 if(currentUser!==admin) return;
 let name=wName.value;
 let price=Number(wPrice.value);
 if(!name||price<=0){alert("Düzgün daxil et"); return;}
 wholesale.push({name,price});
 wName.value=""; wPrice.value="";
 save();
 loadWholesale();
 loadMarket();
 loadAdminPanel();
}

function loadAdminPanel(){
 if(currentUser!==admin) return;
 adminWholesale.innerHTML="<h4>Topdan mallar:</h4>";
 wholesale.forEach((i,idx)=>{
   adminWholesale.innerHTML+=`<div class="item">${i.name} | Qiymət: ${i.price} AZN
   <button onclick="deleteWholesale(${idx})">Sil</button></div>`;
 });

 // İstifadəçilər siyahısı
 adminUsers.innerHTML="<ul>";
 Object.keys(users).forEach(u=>{
     adminUsers.innerHTML+=`<li>${u} - Pul: ${users[u].money.toFixed(2)} AZN | Məhsullar: ${users[u].stock.length}</li>`;
 });
 adminUsers.innerHTML+="</ul>";

 adminMoneyDisplay.innerText = adminMoney.toFixed(2);
}

// --- PAGE LOAD ---
startPage();
function startPage(){
 let savedUsers = localStorage.getItem("users");
 if(savedUsers){
     users = JSON.parse(savedUsers);
     wholesale = JSON.parse(localStorage.getItem("wholesale")) || [];
     market = JSON.parse(localStorage.getItem("market")) || [];
     admin = localStorage.getItem("admin") || null;
     adminMoney = Number(localStorage.getItem("adminMoney")) || 999999999;
     currentUser = localStorage.getItem("currentUser");
     if(currentUser && users[currentUser]) start();
 }
}
</script>
</body>
</html>
