<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Gerador de Tabela de Logaritmos</title>

<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>

<style>
:root{font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif}
body{margin:0;background:#f3f4f6;color:#111}

/* ===== LOGIN ===== */
#login-screen{
  position:fixed;
  inset:0;
  display:flex;
  justify-content:center;
  align-items:center;
  background:#000;
  z-index:999;
}
#login-box{
  background:#0a0a0a;
  padding:30px;
  width:360px;
  border-radius:12px;
  box-shadow:0 0 30px #00f7ff;
  color:#0ff;
  text-align:center;
}
#login-box input{
  width:90%;
  padding:10px;
  margin:8px 0;
  border-radius:6px;
  border:none;
  background:#111;
  color:#0ff;
}
#login-box button{
  width:95%;
  padding:10px;
  background:#00eaff;
  border:none;
  border-radius:6px;
  font-weight:bold;
  cursor:pointer;
}

/* ===== TOPO ===== */
.topbar{
  background:#000;
  color:#0ff;
  padding:10px 20px;
  display:flex;
  justify-content:space-between;
  align-items:center;
}
.topbar button{
  background:none;
  border:none;
  color:#0ff;
  font-size:16px;
  cursor:pointer;
  margin-left:10px;
}

/* ===== SISTEMA ===== */
#main{display:none}
.container{
  max-width:900px;
  margin:40px auto;
  padding:24px;
  background:white;
  border-radius:12px;
  box-shadow:0 6px 24px rgba(2,6,23,0.08)
}
h1{margin:0 0 12px;font-size:20px}
label{display:block;margin-top:10px;font-size:13px}
input,select{
  width:100%;
  padding:8px;
  margin-top:6px;
  border-radius:6px;
  border:1px solid #d1d5db
}
.row{display:flex;gap:12px}
.row>*{flex:1}
button{
  margin-top:14px;
  padding:10px 14px;
  border-radius:8px;
  border:0;
  background:#2563eb;
  color:white;
  font-weight:600;
  cursor:pointer
}
table{
  width:100%;
  border-collapse:collapse;
  margin-top:18px
}
th,td{
  padding:6px 8px;
  border-bottom:1px solid #e5e7eb;
  text-align:right
}
th{text-align:left}

/* ===== ADMIN ===== */
#admin-panel{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.95);
  display:none;
  z-index:1000;
}
.admin-box{
  max-width:500px;
  margin:40px auto;
  background:#0a0a0a;
  padding:20px;
  border-radius:12px;
  color:#0ff;
}
.user-card{
  background:#111;
  padding:10px;
  border-radius:8px;
  margin:8px 0;
}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="login-screen">
  <div id="login-box">
    <h2>🔐 Login</h2>
    <input id="user" placeholder="Usuário">
    <input id="pass" type="password" placeholder="Senha">
    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- SISTEMA -->
<div id="main">
  <div class="topbar">
    <span>📊 Gerador de Logaritmos</span>
    <div>
      <button id="adminBtn" onclick="abrirAdmin()" style="display:none">⚙️ Admin</button>
      <button onclick="sair()">🔓 Sair</button>
    </div>
  </div>

  <div class="container">
    <h1>Gerador de Tabela de Logaritmos</h1>

    <div class="row">
      <div>
        <label>Início</label>
        <input id="start" type="number" value="1">
      </div>
      <div>
        <label>Fim</label>
        <input id="end" type="number" value="10">
      </div>
    </div>

    <label>Base</label>
    <select id="base">
      <option value="10">10</option>
      <option value="e">e</option>
      <option value="2">2</option>
    </select>

    <label>Casas decimais</label>
    <input id="precision" type="number" value="3">

    <button onclick="gerar()">Gerar</button>
    <button onclick="baixar()">Baixar Excel</button>

    <div id="preview"></div>
  </div>
</div>

<!-- ADMIN -->
<div id="admin-panel">
  <div class="admin-box">
    <button onclick="fecharAdmin()">❌</button>
    <h2>Painel Admin</h2>
    <div id="listaUsuarios"></div>
    <input id="novoUser" placeholder="Novo usuário">
    <input id="novoPass" placeholder="Senha">
    <button onclick="criarUsuario()">Criar usuário</button>
  </div>
</div>

<script>
/* ===== BANCO ===== */
let banco = JSON.parse(localStorage.getItem("usuarios")) || {};
if(!banco.CLX){
  banco.CLX = {senha:"02072007", bloqueado:false};
  localStorage.setItem("usuarios", JSON.stringify(banco));
}

/* ===== LOGIN AUTO ===== */
document.addEventListener("DOMContentLoaded", ()=>{
  const logado = localStorage.getItem("logado");
  if(logado){
    login-screen.style.display="none";
    main.style.display="block";
    if(logado==="CLX") adminBtn.style.display="inline";
  }
});

/* ===== LOGIN ===== */
function login(){
  const u = user.value.trim();
  const p = pass.value;
  banco = JSON.parse(localStorage.getItem("usuarios"));

  if(!banco[u]) return alert("Usuário não existe");
  if(banco[u].senha!==p) return alert("Senha incorreta");
  if(banco[u].bloqueado) return alert("Usuário bloqueado");

  localStorage.setItem("logado",u);
  login-screen.style.display="none";
  main.style.display="block";

  if(u==="CLX") adminBtn.style.display="inline";
}

/* ===== SAIR ===== */
function sair(){
  localStorage.removeItem("logado");
  location.reload();
}

/* ===== GERADOR ===== */
function gerar(){
  let s=+start.value, e=+end.value;
  let b=base.value==="e"?Math.E:+base.value;
  let p=+precision.value;
  let html="<table><tr><th>n</th><th>log</th></tr>";
  for(let i=s;i<=e;i++){
    html+=`<tr><td>${i}</td><td>${(Math.log(i)/Math.log(b)).toFixed(p)}</td></tr>`;
  }
  html+="</table>";
  preview.innerHTML=html;
}
function baixar(){
  gerar();
  const rows=[...preview.querySelectorAll("tr")]
    .slice(1).map(r=>({n:r.cells[0].innerText,log:r.cells[1].innerText}));
  const ws=XLSX.utils.json_to_sheet(rows);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"log");
  XLSX.writeFile(wb,"logaritmos.xlsx");
}

/* ===== ADMIN ===== */
function abrirAdmin(){
  atualizarUsuarios();
  admin-panel.style.display="block";
}
function fecharAdmin(){
  admin-panel.style.display="none";
}
function atualizarUsuarios(){
  banco=JSON.parse(localStorage.getItem("usuarios"));
  listaUsuarios.innerHTML="";
  for(let u in banco){
    listaUsuarios.innerHTML+=`
      <div class="user-card">
        ${u}
        ${u!=="CLX"?`<button onclick="excluirUsuario('${u}')">Excluir</button>`:""}
      </div>`;
  }
}
function criarUsuario(){
  banco[novoUser.value]={senha:novoPass.value,bloqueado:false};
  localStorage.setItem("usuarios",JSON.stringify(banco));
  atualizarUsuarios();
}
function excluirUsuario(u){
  delete banco[u];
  localStorage.setItem("usuarios",JSON.stringify(banco));
  atualizarUsuarios();
}
</script>
</body>
</html>
