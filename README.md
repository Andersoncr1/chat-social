<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chat Social Inteligente - Versão Final</title>
<style>
body { font-family: 'Arial', sans-serif; margin:0; background: linear-gradient(to right, #ff6a6a, #ffb347); color:#fff; }
header { background: #e91e63; padding:20px; text-align:center; font-size:24px; font-weight:bold; color:#fff; text-shadow:1px 1px 3px #000; }
.container { padding:20px; display:flex; flex-direction:column; height:calc(100vh - 80px); }
.hidden { display:none; }
input, button { padding:10px; margin:5px 0; border-radius:8px; border:none; font-size:16px; }
button { background:#ff4081; color:#fff; font-weight:bold; cursor:pointer; transition:0.3s; }
button:hover { background:#e91e63; }
.main-chat-container { display:flex; flex:1; gap:10px; }
.user-list { width:200px; border:2px solid #fff; border-radius:10px; padding:10px; background: rgba(0,0,0,0.2); overflow-y:auto; }
.user-list span { display:flex; align-items:center; background:#4caf50; padding:5px 10px; border-radius:8px; margin-bottom:5px; cursor:pointer; }
.user-list img { width:32px; height:32px; border-radius:50%; margin-right:6px; }
.chat { flex:1; border:2px solid #fff; border-radius:10px; padding:12px; background: rgba(0,0,0,0.2); display:flex; flex-direction:column; overflow-y:auto; }
.msg { margin:6px 0; padding:6px 10px; border-radius:10px; max-width:80%; word-break: break-word; }
.me { background: #4caf50; color:#fff; margin-left:auto; text-align:right; }
.bot { background: #ff9800; color:#fff; margin-right:auto; text-align:left; }
.chat-controls { display:flex; gap:10px; align-items:center; margin-top:10px; }
.credits { font-weight:bold; font-size:16px; color:#fff; }
.recharge-box { display:none; background: rgba(0,0,0,0.3); padding:10px; border-radius:8px; font-size:14px; position:absolute; top:10%; right:10%; z-index:10; }
.admin-panel { background: rgba(0,0,0,0.3); padding:15px; border-radius:10px; overflow-y:auto; max-height:80vh; }
.admin-panel table { width:100%; border-collapse:collapse; }
.admin-panel th, .admin-panel td { border:1px solid #fff; padding:8px; text-align:center; }
.admin-panel th { background:#ff4081; }
.admin-panel td { background: rgba(255,255,255,0.2); }
.admin-panel input { margin-bottom:10px; padding:8px; border-radius:6px; border:none; width:100%; }
</style>
</head>
<body>
<header>Chat Social Inteligente 💖</header>
<div class="container">

<!-- LOGIN / CADASTRO CLIENTE -->
<div id="login">
  <h3>Login / Cadastro</h3>
  <div id="loginFields">
    <input id="loginUser" placeholder="Usuário" />
    <input id="loginPass" placeholder="Senha" type="password" />
    <button onclick="loginUser()">Entrar</button>
  </div>
  <hr>
  <div id="registerFields">
    <h4>Cadastro</h4>
    <input id="regFirstName" placeholder="Nome" />
    <input id="regLastName" placeholder="Sobrenome" />
    <input id="regAge" placeholder="Idade" type="number" />
    <input id="regUsername" placeholder="Escolha um usuário" />
    <input id="regPassword" placeholder="Escolha uma senha" type="password" />
    <input type="file" id="regProfilePic" accept="image/*" />
    <button onclick="registerUser()">Finalizar Cadastro</button>
  </div>
</div>

<!-- CHAT CLIENTE -->
<div id="chatArea" class="hidden" style="display:flex; flex-direction:column; height:100%; position:relative;">
  <div class="credits" id="credits"></div>
  <div class="main-chat-container">
    <div class="user-list" id="onlineUsers">Usuários Online:</div>
    <div class="chat" id="chat"></div>
  </div>
  <div class="chat-controls">
    <input id="msg" placeholder="Digite sua mensagem" style="flex:1;" />
    <input type="file" id="fileInput" accept="image/*" />
    <button onclick="sendMsg()">Enviar (1 ponto)</button>
    <button onclick="sendPhoto()">Enviar Foto (5 pontos)</button>
    <button onclick="toggleRechargeBox()">Recarregar</button>
    <div id="rechargeBox" class="recharge-box">
      <p>Chave Pix: <strong>5e358183-1e1b-4eb3-9d79-133ddb5376bd</strong></p>
      <p>Valor mínimo: R$5</p>
    </div>
  </div>
</div>

<!-- PAINEL ADMIN -->
<div id="adminLogin">
  <h3>Admin Login</h3>
  <input id="adminUser" placeholder="Usuário admin" />
  <input id="adminPass" placeholder="Senha admin" type="password" />
  <button onclick="adminLogin()">Entrar</button>
</div>
<div id="adminPanel" class="hidden admin-panel">
  <h4>Painel Admin - Apenas Visível ao Administrador</h4>
  <input id="searchUser" placeholder="Localizar usuário" onkeyup="filterUserTable()" />
  <table id="userTable">
    <tr><th>Nome</th><th>Sobrenome</th><th>Usuário</th><th>Idade</th><th>Créditos</th><th>Status</th><th>Ações</th><th>Histórico Recargas</th></tr>
  </table>
</div>

<script>
let currentUser=null;
let currentChatUser=null;
const adminUserReal='admin';
const adminPassReal='admin123';
const users=JSON.parse(localStorage.getItem('users'))||[];
const rechargeHistory=JSON.parse(localStorage.getItem('rechargeHistory'))||[];

function saveUsers(){ localStorage.setItem('users',JSON.stringify(users)); }
function saveRechargeHistory(){ localStorage.setItem('rechargeHistory',JSON.stringify(rechargeHistory)); }
function getAvatar(user){ return user.profilePic||`https://via.placeholder.com/32/4D96FF/fff?text=${user.firstName[0].toUpperCase()}`; }

function updateOnlineUsers(){
  const onlineDiv=document.getElementById('onlineUsers');
  onlineDiv.innerHTML='Usuários Online:' + users.filter(u=>u.online && u.username!==currentUser.username).map(u=>`<span onclick="startChat('${u.username}')"><img src='${getAvatar(u)}'/>${u.firstName}</span>`).join(' ');
}

function startChat(username){
  currentChatUser=users.find(u=>u.username===username);
  document.getElementById('chat').innerHTML=`<div class='msg bot'>Você iniciou conversa com ${currentChatUser.firstName}</div>`;
}

function registerUser(){
  const firstName=document.getElementById('regFirstName').value;
  const lastName=document.getElementById('regLastName').value;
  const age=document.getElementById('regAge').value;
  const username=document.getElementById('regUsername').value;
  const password=document.getElementById('regPassword').value;
  const fileInput=document.getElementById('regProfilePic');
  if(!firstName||!lastName||!age||!username||!password) return alert('Preencha todos os campos');
  if(users.find(u=>u.username===username)) return alert('Usuário já existe');
  let profilePic=null;
  if(fileInput.files.length>0){
    const reader=new FileReader();
    reader.onload=function(e){ profilePic=e.target.result; finalizeRegistration(firstName,lastName,age,username,password,profilePic); };
    reader.readAsDataURL(fileInput.files[0]);
  } else finalizeRegistration(firstName,lastName,age,username,password,profilePic);
}

function finalizeRegistration(firstName,lastName,age,username,password,profilePic){
  const newUser={firstName,lastName,age,username,password,credits:100,online:false,profilePic};
  users.push(newUser); saveUsers();
  alert('Cadastro concluído com sucesso! Você já pode fazer login com usuário e senha.');
  document.getElementById('regFirstName').value='';
  document.getElementById('regLastName').value='';
  document.getElementById('regAge').value='';
  document.getElementById('regUsername').value='';
  document.getElementById('regPassword').value='';
  document.getElementById('regProfilePic').value='';
}

function loginUser(){
  const user=document.getElementById('loginUser').value;
  const pass=document.getElementById('loginPass').value;
  const found=users.find(u=>u.username===user && u.password===pass);
  if(!found) return alert('Usuário ou senha incorretos');
  currentUser=found; currentUser.online=true; saveUsers();
  document.getElementById('login').classList.add('hidden');
  document.getElementById('chatArea').classList.remove('hidden');
  updateCredits(); updateOnlineUsers();
  window.addEventListener('beforeunload',()=>{ currentUser.online=false; saveUsers(); });
}

function updateCredits(){ if(currentUser) document.getElementById('credits').innerText='Créditos: '+currentUser.credits; }
function sendMsg(){ const text=document.getElementById('msg').value; if(!text) return;
  if(currentUser.credits<=0) return alert('Créditos esgotados');
  currentUser.credits--; updateCredits(); saveUsers();
  document.getElementById('chat').innerHTML+=`<div class='msg me'>${text}</div>`;
  document.getElementById('msg').value='';
}
function sendPhoto(){ const fileInput=document.getElementById('fileInput'); if(fileInput.files.length===0) return alert('Selecione uma foto');
  if(currentUser.credits<5) return alert('Créditos insuficientes');
  const reader=new FileReader(); reader.onload=function(e){ currentUser.credits-=5; updateCredits(); saveUsers(); document.getElementById('chat').innerHTML+=`<div class='msg me'><img src='${e.target.result}' style='max-width:80%; border-radius:10px;' /></div>`; };
  reader.readAsDataURL(fileInput.files[0]);
}
function toggleRechargeBox(){ const box=document.getElementById('rechargeBox');
  if(box.style.display==='none'||box.style.display===''){ box.style.display='block';
    const amount=prompt('Digite o valor do crédito a comprar (mínimo R$5)');
    if(amount && parseFloat(amount)>=5){ const points=parseFloat(amount)*20; currentUser.credits+=points; updateCredits();
      rechargeHistory.push({user:currentUser.username,valor:amount,pontos:points,data:new Date().toLocaleString()}); saveUsers(); saveRechargeHistory(); alert('Créditos adicionados!');
    }
  } else box.style.display='none';
}

function adminLogin(){
  const user=document.getElementById('adminUser').value;
  const pass=document.getElementById('adminPass').value;
  if(user===adminUserReal && pass===adminPassReal){
    document.getElementById('adminLogin').classList.add('hidden');
    document.getElementById('adminPanel').classList.remove('hidden');
    refreshUserTable();
  } else alert('Usuário ou senha admin incorretos');
}

function refreshUserTable(){
  const table=document.getElementById('userTable'); table.innerHTML='<tr><th>Nome</th><th>Sobrenome</th><th>Usuário</th><th>Idade</th><th>Créditos</th><th>Status</th><th>Ações</th><th>Histórico Recargas</th></tr>';
  users.forEach(u=>{
    const row=table.insertRow(); row.insertCell(0).innerText=u.firstName; row.insertCell(1).innerText=u.lastName;
    row.insertCell(2).innerText=u.username; row.insertCell(3).innerText=u.age; row.insertCell(4).innerText=u.credits; row.insertCell(5).innerText=u.online?'Online':'Offline';
    const actionCell=row.insertCell(6); const addBtn=document.createElement('button'); addBtn.innerText='+100 créditos';
    addBtn.onclick=()=>{ u.credits+=100; saveUsers(); refreshUserTable(); if(currentUser&&u.username===currentUser.username) updateCredits(); };
    actionCell.appendChild(addBtn);
    const historyCell=row.insertCell(7);
    historyCell.innerText=rechargeHistory.filter(r=>r.user===u.username).map(r=>`[${r.data}] R$${r.valor} -> ${r.pontos} pontos`).join(' | ');
  });
}

function filterUserTable(){
  const filter=document.getElementById('searchUser').value.toUpperCase();
  const table=document.getElementById('userTable'); const tr=table.getElementsByTagName('tr');
  for(let i=1;i<tr.length;i++){ tr[i].style.display=(tr[i].cells[2].innerText.toUpperCase().indexOf(filter)>-1)?'':'none'; }
}
</script>
</body>
</html>
