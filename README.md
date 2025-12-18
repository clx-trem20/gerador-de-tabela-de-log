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
  background:rgba(0,0,0,.95);
  display:none; z-index:2000;
}
.adminBox{
  max-width:500px;
  margin:40px auto;
  background:#0a0a0a; color:#0ff;
  padding:20px;
  border-radius:12px;
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
  </div>
</div>

<!-- ADMIN -->
<div id="adminPanel">
  <div class="adminBox">
    <button onclick="fecharAdmin()">❌</button>
    <h3>⚙️ Painel Admin</h3>

    <input id="novaSenha" type="password" placeholder="Nova senha">
    <button onclick="trocarSenha()">Salvar nova senha</button>

    <p style="font-size:12px;margin-top:10px">
      🔒 Senha protegida com hash (SHA-256)
    </p>
  </div>
</div>

<!-- FIREBASE + LÓGICA -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, doc, getDoc, updateDoc, addDoc, collection, serverTimestamp }
from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCq7EmnBjUTO7WwFQYUERg9huJ4V2VLN8I",
  authDomain: "gerado-de-log.firebaseapp.com",
  projectId: "gerado-de-log",
  storageBucket: "gerado-de-log.firebasestorage.app",
  messagingSenderId: "884275714766",
  appId: "1:884275714766:web:616fb069a0db82eb880e9f"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

/* ===== HASH ===== */
async function hashSenha(txt){
  const data = new TextEncoder().encode(txt);
  const hash = await crypto.subtle.digest("SHA-256", data);
  return Array.from(new Uint8Array(hash))
    .map(b=>b.toString(16).padStart(2,"0")).join("");
}

/* ===== LOGIN ===== */
window.login = async function(){
  const ref = doc(db,"config","admin");
  const snap = await getDoc(ref);

  if(!snap.exists()){
    alert("Configuração não encontrada no banco");
    return;
  }

  const dados = snap.data();
  const senhaHash = await hashSenha(pass.value);

  if(user.value === dados.usuario && senhaHash === dados.senha){
    localStorage.setItem("logado","1");

    loginScreen.style.display="none";
    main.style.display="block";
    adminBtn.style.display="inline";
  }else{
    alert("Usuário ou senha incorretos");
  }
};

/* ===== TROCAR SENHA ===== */
window.trocarSenha = async function(){
  if(novaSenha.value.length < 3){
    alert("Senha muito curta");
    return;
  }
  const novaHash = await hashSenha(novaSenha.value);
  await updateDoc(doc(db,"config","admin"),{senha:novaHash});
  alert("Senha alterada com sucesso");
  novaSenha.value="";
};

/* ===== HISTÓRICO ===== */
window.salvarHistorico = async function(dados){
  await addDoc(collection(db,"historico"),{
    ...dados,
    data: serverTimestamp()
  });
};

/* ===== CONTROLE ===== */
window.sair = function(){
  localStorage.removeItem("logado");
  location.reload();
};
window.abrirAdmin = ()=> adminPanel.style.display="block";
window.fecharAdmin = ()=> adminPanel.style.display="none";
</script>

<script>
function gerar(){
  let s=+start.value, e=+end.value;
  let b=base.value==="e"?Math.E:+base.value;
  let p=+precision.value;

  let html="<table><tr><th>n</th><th>log</th></tr>";
  let hist=[];

  for(let i=s;i<=e;i++){
    const v=(Math.log(i)/Math.log(b)).toFixed(p);
    html+=`<tr><td>${i}</td><td>${v}</td></tr>`;
    hist.push({n:i,log:v});
  }

  preview.innerHTML=html;
  salvarHistorico({inicio:s,fim:e,base:b,resultado:hist});
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
</script>

</body>
</html>
