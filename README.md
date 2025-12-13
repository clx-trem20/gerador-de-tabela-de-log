<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Gerador de Logaritmos CLX</title>

<style>
:root{font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif}
body{margin:0;background:#f3f4f6;color:#111}

/* LOGIN */
#login-screen{
  position:fixed;
  inset:0;
  background:#111827;
  display:flex;
  align-items:center;
  justify-content:center;
  z-index:999;
}
.login-box{
  background:#fff;
  padding:30px;
  border-radius:12px;
  width:320px;
  box-shadow:0 10px 30px rgba(0,0,0,.4);
}
.login-box h2{text-align:center}
.login-box input{
  width:100%;
  padding:10px;
  margin:8px 0;
  border-radius:8px;
  border:1px solid #ccc;
}
.login-box button{
  width:100%;
  padding:10px;
  background:#2563eb;
  color:#fff;
  border:none;
  border-radius:8px;
  cursor:pointer;
  font-weight:600;
}

/* TOPO */
.topbar{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:20px;
}
.topbar button{
  background:#111827;
  color:#fff;
}

/* SITE */
#main{display:none}
.container{
  max-width:900px;
  margin:40px auto;
  padding:24px;
  background:#fff;
  border-radius:12px;
  box-shadow:0 6px 24px rgba(2,6,23,.08)
}
.row{display:flex;gap:12px}
.row>*{flex:1}
label{font-size:13px;display:block;margin-top:10px}
input,select{
  width:100%;
  padding:8px;
  margin-top:6px;
  border-radius:6px;
  border:1px solid #d1d5db
}
button{
  margin-top:14px;
  padding:10px;
  border-radius:8px;
  border:0;
  background:#2563eb;
  color:#fff;
  font-weight:600;
  cursor:pointer
}
table{width:100%;border-collapse:collapse;margin-top:18px}
th,td{padding:6px;border-bottom:1px solid #e5e7eb;text-align:right}
th{text-align:left}

/* FOOTER */
footer{
  margin-top:30px;
  text-align:center;
  font-size:14px;
  color:#6b7280;
  opacity:0;
  animation:fadeUp 1.2s ease forwards;
}
footer strong{color:#2563eb}
@keyframes fadeUp{
  from{opacity:0;transform:translateY(10px)}
  to{opacity:1;transform:translateY(0)}
}

/* ADMIN */
#admin-panel{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.8);
  display:none;
}
.admin-box{
  background:#fff;
  max-width:400px;
  margin:60px auto;
  padding:20px;
  border-radius:12px;
}
.user-card{
  border:1px solid #ddd;
  padding:8px;
  margin:6px 0;
  border-radius:6px;
}
</style>

<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>

<body>

<!-- LOGIN -->
<div id="login-screen">
  <div class="login-box">
    <h2>🔐 Login</h2>
    <input id="user" placeholder="Usuário">
    <input id="pass" type="password" placeholder="Senha">
    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- SITE -->
<div id="main">
  <div class="container">

    <div class="topbar">
      <strong>Gerador de Logaritmos</strong>
      <div>
        <button onclick="abrirAdmin()">Admin</button>
        <button onclick="sair()">Sair</button>
      </div>
    </div>

    <div class="row">
      <div>
        <label>Início</label>
        <input id="start" type="number" value="1" min="1">
      </div>
      <div>
        <label>Fim</label>
        <input id="end" type="number" value="10" min="1">
      </div>
    </div>

    <label>Base</label>
    <select id="base">
      <option value="10">10</option>
      <option value="e">e</option>
      <option value="2">2</option>
      <option value="other">Outra</option>
    </select>

    <div id="otherBaseWrap" style="display:none">
      <label>Digite a base</label>
      <input id="otherBase" type="number" step="any" value="3">
    </div>

    <label>Casas decimais</label>
    <input id="precision" type="number" value="3" min="0" max="8">

    <div class="row">
      <button onclick="gerar()">Gerar</button>
      <button onclick="baixar()">Excel</button>
    </div>

    <div class="preview" id="preview"></div>

    <footer>
      ✨ Criado por <strong>CLX</strong> © <span id="ano"></span>
    </footer>

  </div>
</div>

<!-- ADMIN -->
<div id="admin-panel">
  <div class="admin-box">
    <h3>Painel Admin</h3>
    <div id="listaUsuarios"></div>
    <input id="novoUser" placeholder="Novo usuário">
    <input id="novoPass" placeholder="Senha">
    <button onclick="criarUsuario()">Criar</button>
    <button onclick="fecharAdmin()">Fechar</button>
  </div>
</div>

<script>
document.getElementById("ano").textContent = new Date().getFullYear();

/* USUÁRIOS */
let usuarios = JSON.parse(localStorage.getItem("usuarios")) || {
  CLX: {senha:"02072007"}
};
localStorage.setItem("usuarios", JSON.stringify(usuarios));

/* LOGIN */
function login(){
  const u=user.value,p=pass.value;
  if(usuarios[u] && usuarios[u].senha===p){
    localStorage.setItem("logado",u);
    login-screen.style.display="none";
    main.style.display="block";
  } else alert("Login inválido");
}

/* AUTOLOGIN */
if(localStorage.getItem("logado")){
  login-screen.style.display="none";
  main.style.display="block";
}

/* SAIR */
function sair(){
  localStorage.removeItem("logado");
  location.reload();
}

/* BASE */
base.onchange=()=>otherBaseWrap.style.display=base.value==="other"?"block":"none";

/* GERAR */
function gerar(){
  let b;
  if(base.value==="10") b=10;
  else if(base.value==="2") b=2;
  else if(base.value==="e") b=Math.E;
  else b=parseFloat(otherBase.value);

  if(!(b>0)||b===1){alert("Base inválida");return;}

  let html="<table><tr><th>n</th><th>log</th></tr>";
  for(let i=start.value;i<=end.value;i++){
    html+=`<tr><td>${i}</td><td>${(Math.log(i)/Math.log(b)).toFixed(precision.value)}</td></tr>`;
  }
  html+="</table>";
  preview.innerHTML=html;
}

/* EXCEL */
function baixar(){
  const rows=[];
  for(let i=start.value;i<=end.value;i++){
    rows.push({n:i,log:(Math.log(i)/Math.log(base.value==="e"?Math.E:base.value==="other"?otherBase.value:base.value)).toFixed(precision.value)});
  }
  const ws=XLSX.utils.json_to_sheet(rows);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"log");
  XLSX.writeFile(wb,"tabela_log.xlsx");
}

/* ADMIN */
function abrirAdmin(){
  if(localStorage.getItem("logado")!=="CLX"){alert("Apenas admin");return;}
  atualizarUsuarios();
  admin-panel.style.display="block";
}
function fecharAdmin(){admin-panel.style.display="none";}
function atualizarUsuarios(){
  listaUsuarios.innerHTML="";
  for(let u in usuarios){
    listaUsuarios.innerHTML+=`
      <div class="user-card">
        <b>${u}</b>
        <button onclick="delete usuarios['${u}'];localStorage.setItem('usuarios',JSON.stringify(usuarios));atualizarUsuarios()">Excluir</button>
      </div>`;
  }
}
function criarUsuario(){
  usuarios[novoUser.value]={senha:novoPass.value};
  localStorage.setItem("usuarios",JSON.stringify(usuarios));
  atualizarUsuarios();
}
</script>

</body>
</html>
