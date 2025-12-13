<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Gerador de Logaritmos CLX</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#f3f4f6;
}

/* LOGIN */
#loginScreen{
  position:fixed;
  inset:0;
  background:#111827;
  display:flex;
  align-items:center;
  justify-content:center;
}

.loginBox{
  background:#fff;
  padding:24px;
  border-radius:12px;
  width:280px;
}

/* SISTEMA */
#mainScreen{display:none}

.container{
  max-width:900px;
  margin:40px auto;
  background:#fff;
  padding:24px;
  border-radius:12px;
}

.topbar{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:16px;
}

button{
  padding:8px 14px;
  border:0;
  border-radius:8px;
  background:#2563eb;
  color:#fff;
  cursor:pointer;
}

input,select{
  width:100%;
  padding:8px;
  margin:6px 0 12px;
}

#otherBaseWrap{display:none}

table{
  width:100%;
  border-collapse:collapse;
}
td,th{
  border-bottom:1px solid #ddd;
  padding:6px;
}

/* ADMIN */
#adminPanel{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.7);
  display:none;
  align-items:center;
  justify-content:center;
}

.adminBox{
  background:#fff;
  padding:20px;
  border-radius:12px;
  width:320px;
}

.userCard{
  border:1px solid #ddd;
  padding:6px;
  margin:6px 0;
}

footer{
  margin-top:20px;
  text-align:center;
  opacity:0;
  animation:fade .8s forwards;
}

@keyframes fade{
  to{opacity:1}
}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="loginScreen">
  <div class="loginBox">
    <h3>🔐 Login</h3>
    <input id="loginUser" placeholder="Usuário">
    <input id="loginPass" type="password" placeholder="Senha">
    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- SISTEMA -->
<div id="mainScreen">
  <div class="container">

    <div class="topbar">
      <strong>Gerador de Logaritmos</strong>
      <div>
        <button onclick="abrirAdmin()">Admin</button>
        <button onclick="logout()">Sair</button>
      </div>
    </div>

    <label>Base do log</label>
    <select id="base">
      <option value="10">10</option>
      <option value="2">2</option>
      <option value="e">e</option>
      <option value="other">Outra</option>
    </select>

    <div id="otherBaseWrap">
      <input id="otherBase" placeholder="Digite a base">
    </div>

    <button onclick="gerar()">Gerar</button>

    <div class="preview" id="preview"></div>

    <footer>
      ✨ Criado por <strong>CLX</strong> © <span id="ano"></span>
    </footer>
  </div>
</div>

<!-- ADMIN -->
<div id="adminPanel">
  <div class="adminBox">
    <h3>Painel Admin</h3>
    <div id="listaUsuarios"></div>

    <input id="novoUser" placeholder="Novo usuário">
    <input id="novoPass" placeholder="Senha">
    <button onclick="criarUsuario()">Criar usuário</button>
    <button onclick="fecharAdmin()">Fechar</button>
  </div>
</div>

<script>
/* ====== AUTH ====== */
const loginScreen = document.getElementById("loginScreen");
const mainScreen  = document.getElementById("mainScreen");

let usuarios = JSON.parse(localStorage.getItem("usuarios")) || {
  CLX:{senha:"1234",admin:true,bloqueado:false}
};
localStorage.setItem("usuarios",JSON.stringify(usuarios));

function login(){
  const u = loginUser.value;
  const p = loginPass.value;

  if(!usuarios[u]) return alert("Usuário não existe");
  if(usuarios[u].bloqueado) return alert("Usuário bloqueado");
  if(usuarios[u].senha !== p) return alert("Senha incorreta");

  localStorage.setItem("logado",u);
  loginScreen.style.display="none";
  mainScreen.style.display="block";
}

function logout(){
  localStorage.removeItem("logado");
  location.reload();
}

if(localStorage.getItem("logado")){
  loginScreen.style.display="none";
  mainScreen.style.display="block";
}

/* ====== ADMIN ====== */
const adminPanel = document.getElementById("adminPanel");

function abrirAdmin(){
  const u = localStorage.getItem("logado");
  if(!usuarios[u].admin) return alert("Sem permissão");
  atualizarUsuarios();
  adminPanel.style.display="flex";
}

function fecharAdmin(){
  adminPanel.style.display="none";
}

function atualizarUsuarios(){
  listaUsuarios.innerHTML="";
  for(let u in usuarios){
    listaUsuarios.innerHTML+=`
      <div class="userCard">
        <b>${u}</b> ${usuarios[u].bloqueado?"(Bloqueado)":""}<br>
        <button onclick="bloquear('${u}')">Bloquear</button>
        <button onclick="trocarSenha('${u}')">Trocar senha</button>
      </div>`;
  }
}

function criarUsuario(){
  usuarios[novoUser.value]={
    senha:novoPass.value,
    admin:false,
    bloqueado:false
  };
  localStorage.setItem("usuarios",JSON.stringify(usuarios));
  atualizarUsuarios();
}

function bloquear(u){
  usuarios[u].bloqueado=!usuarios[u].bloqueado;
  localStorage.setItem("usuarios",JSON.stringify(usuarios));
  atualizarUsuarios();
}

function trocarSenha(u){
  const n = prompt("Nova senha:");
  if(n){
    usuarios[u].senha=n;
    localStorage.setItem("usuarios",JSON.stringify(usuarios));
  }
}

/* ====== APP ====== */
document.getElementById("ano").textContent = new Date().getFullYear();

base.addEventListener("change",()=>{
  otherBaseWrap.style.display = base.value==="other" ? "block" : "none";
});

function gerar(){
  let b;
  if(base.value==="10") b=10;
  else if(base.value==="2") b=2;
  else if(base.value==="e") b=Math.E;
  else b=parseFloat(otherBase.value);

  if(!(b>0) || b===1) return alert("Base inválida");

  let html="<table><tr><th>n</th><th>log</th></tr>";
  for(let i=1;i<=10;i++){
    html+=`<tr><td>${i}</td><td>${(Math.log(i)/Math.log(b)).toFixed(3)}</td></tr>`;
  }
  html+="</table>";
  preview.innerHTML=html;
}
</script>

</body>
</html>
