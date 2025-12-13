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
input,select{width:100%;padding:8px;margin-top:6px;border-radius:6px;border:1px solid #d1d5db}
button{margin-top:14px;padding:10px;border-radius:8px;border:0;background:#2563eb;color:#fff;font-weight:600;cursor:pointer}
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
      <option value="other">Outra</option>
    </select>

    <div id="otherBaseWrap" style="display:none">
      <input id="otherBase" value="3">
    </div>

    <label>Precisão</label>
    <input id="precision" value="3">

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

<script>
/* ANO AUTOMÁTICO */
document.getElementById("ano").textContent = new Date().getFullYear();

/* USUÁRIOS */
const usuarios = {
  CLX: "02072007"
};

/* LOGIN */
function login(){
  const u = document.getElementById("user").value;
  const p = document.getElementById("pass").value;

  if(usuarios[u] && usuarios[u] === p){
    localStorage.setItem("logado", u);
    document.getElementById("login-screen").style.display = "none";
    document.getElementById("main").style.display = "block";
  } else {
    alert("Usuário ou senha incorretos");
  }
}

/* AUTO LOGIN */
if(localStorage.getItem("logado")){
  document.getElementById("login-screen").style.display = "none";
  document.getElementById("main").style.display = "block";
}

/* GERADOR */
function gerar(){
  let base = document.getElementById("base").value;
  if(base==="e") base = Math.E;
  if(base==="other") base = document.getElementById("otherBase").value;

  let html="<table><tr><th>n</th><th>log</th></tr>";
  for(let i=start.value;i<=end.value;i++){
    html+=`<tr><td>${i}</td><td>${(Math.log(i)/Math.log(base)).toFixed(precision.value)}</td></tr>`;
  }
  html+="</table>";
  preview.innerHTML = html;
}

function baixar(){
  const rows=[];
  for(let i=start.value;i<=end.value;i++){
    rows.push({n:i,log:(Math.log(i)/Math.log(base.value==="e"?Math.E:base.value==="other"?otherBase.value:base.value)).toFixed(precision.value)});
  }
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "log");
  XLSX.writeFile(wb, "tabela_log.xlsx");
}
</script>

</body>
</html>
