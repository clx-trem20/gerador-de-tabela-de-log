<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Gerador de Logaritmos</title>

<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>

<style>
body{margin:0;font-family:Arial;background:#f3f4f6}

/* LOGIN */
#loginScreen{
  position:fixed; inset:0;
  display:flex; justify-content:center; align-items:center;
  background:#000; z-index:1000;
}
#loginBox{
  background:#0a0a0a; color:#0ff;
  padding:30px; width:320px;
  border-radius:12px; text-align:center;
  box-shadow:0 0 25px #00f7ff;
}
#loginBox input{
  width:90%; padding:10px; margin:8px 0;
  border:none; border-radius:6px;
  background:#111; color:#0ff;
}
#loginBox button{
  width:95%; padding:10px;
  background:#00eaff; border:none;
  border-radius:6px; font-weight:bold;
  cursor:pointer;
}

/* TOPO */
.topbar{
  background:#000; color:#0ff;
  padding:10px 20px;
  display:flex; justify-content:space-between;
}

/* SISTEMA */
#main{display:none}
.container{
  max-width:900px;
  margin:40px auto;
  background:white;
  padding:24px;
  border-radius:12px;
}
.row{display:flex; gap:10px}
.row>*{flex:1}
button{margin-top:10px; padding:10px}
table{width:100%; border-collapse:collapse; margin-top:15px}
th,td{border-bottom:1px solid #ddd; padding:6px}

/* ADMIN */
#adminPanel{
  position:fixed; inset:0;
  background:rgba(0,0,0,0.95);
  display:none; z-index:2000;
}
.adminBox{
  max-width:500px;
  margin:40px auto;
  background:#0a0a0a; color:#0ff;
  padding:20px; border-radius:12px;
}
.userCard{
  background:#111; padding:8px;
  margin:6px 0; border-radius:6px;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="loginScreen">
  <div id="loginBox">
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
    <div class="row">
      <input id="start" type="number" value="1">
      <input id="end" type="number" value="10">
    </div>

    <select id="base">
      <option value="10">Base 10</option>
      <option value="e">Base e</option>
      <option value="2">Base 2</option>
    </select>

    <input id="precision" type="number" value="3">

    <button onclick="gerar()">Gerar</button>
    <button onclick="baixar()">Baixar Excel</button>

    <div id="preview"></div>

  <footer>
      ✨ Criado por <strong>CLX</strong> © <span id="ano"></span>
    </footer>
  </div>
  </div>
</div>

<!-- ADMIN -->
<div id="adminPanel">
  <div class="adminBox">
    <button onclick="fecharAdmin()">❌</button>
    <h3>Painel Admin</h3>
    <div id="listaUsuarios"></div>
    <input id="novoUser" placeholder="Novo usuário">
    <input id="novoPass" placeholder="Senha">
    <button onclick="criarUsuario()">Criar</button>
  </div>
</div>

<script>
// ===== BANCO =====
let banco = JSON.parse(localStorage.getItem("usuarios")) || {};
if(!banco.CLX){
  banco.CLX = {senha:"02072007", bloqueado:false};
  localStorage.setItem("usuarios", JSON.stringify(banco));
}

// ===== AUTO LOGIN =====
document.addEventListener("DOMContentLoaded", ()=>{
  const logado = localStorage.getItem("logado");
  if(logado){
    document.getElementById("loginScreen").style.display="none";
    document.getElementById("main").style.display="block";
    if(logado==="CLX"){
      document.getElementById("adminBtn").style.display="inline";
    }
  }
});

// ===== LOGIN =====
function login(){
  const u = document.getElementById("user").value.trim();
  const p = document.getElementById("pass").value;
  banco = JSON.parse(localStorage.getItem("usuarios"));

  if(!banco[u]) return alert("Usuário não existe");
  if(banco[u].senha !== p) return alert("Senha incorreta");

  localStorage.setItem("logado",u);
  document.getElementById("loginScreen").style.display="none";
  document.getElementById("main").style.display="block";

  if(u==="CLX"){
    document.getElementById("adminBtn").style.display="inline";
  }
}

// ===== SAIR =====
function sair(){
  localStorage.removeItem("logado");
  location.reload();
}

// ===== GERADOR =====
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

// ===== ADMIN =====
function abrirAdmin(){
  atualizarUsuarios();
  document.getElementById("adminPanel").style.display="block";
}
function fecharAdmin(){
  document.getElementById("adminPanel").style.display="none";
}
function atualizarUsuarios(){
  banco = JSON.parse(localStorage.getItem("usuarios"));
  listaUsuarios.innerHTML="";
  for(let u in banco){
    listaUsuarios.innerHTML+=`
      <div class="userCard">
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
