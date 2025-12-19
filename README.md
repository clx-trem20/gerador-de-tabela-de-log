<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Gerador de Logaritmos</title>

<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>

<style>
body{margin:0;font-family:Arial;background:#f3f4f6}
#loginScreen{
  position:fixed; inset:0;
  display:flex; justify-content:center; align-items:center;
  background:#000; z-index:1000;
}
#loginBox{
  background:#0a0a0a; color:#0ff;
  padding:30px; width:320px;
  border-radius:12px; text-align:center;
}
#loginBox input{
  width:90%; padding:10px; margin:8px 0;
}
#loginBox button{width:95%;padding:10px;cursor:pointer}
.topbar{
  background:#000; color:#0ff;
  padding:10px 20px;
  display:flex; justify-content:space-between;
}
#main{display:none}
.container{
  max-width:900px;
  margin:40px auto;
  background:white;
  padding:24px;
  border-radius:12px;
}
#adminPanel{
  position:fixed; inset:0;
  background:rgba(0,0,0,.95);
  display:none; z-index:2000;
}
.adminBox{
  background:#111;color:#0ff;
  max-width:400px;
  margin:40px auto;
  padding:20px;
  border-radius:10px;
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
    <button id="btnLogin">Entrar</button>
  </div>
</div>

<!-- SISTEMA -->
<div id="main">
  <div class="topbar">
    <span>📊 Gerador de Logaritmos</span>
    <div>
      <button id="adminBtn" style="display:none">⚙️ Admin</button>
      <button id="btnSair">🔓 Sair</button>
    </div>
  </div>

  <div class="container">
    <input id="start" type="number" value="1">
    <input id="end" type="number" value="10">
    <select id="base">
      <option value="10">Base 10</option>
      <option value="e">Base e</option>
      <option value="2">Base 2</option>
    </select>
    <input id="precision" type="number" value="3">
    <button onclick="gerar()">Gerar</button>
    <div id="preview"></div>
  </div>
</div>

<!-- ADMIN -->
<div id="adminPanel">
  <div class="adminBox">
    <button id="fecharAdmin">❌</button>
    <h3>Trocar senha</h3>
    <input id="novaSenha" type="password" placeholder="Nova senha">
    <button id="salvarSenha">Salvar</button>
  </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, doc, setDoc, getDoc, updateDoc }
from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

/* FIREBASE */
const firebaseConfig = {
  apiKey: "AIzaSyCq7EmnBjUTO7WwFQYUERg9huJ4V2VLN8I",
  authDomain: "gerado-de-log.firebaseapp.com",
  projectId: "gerado-de-log"
};
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

/* ELEMENTOS */
const loginScreen = document.getElementById("loginScreen");
const main = document.getElementById("main");
const adminBtn = document.getElementById("adminBtn");
const adminPanel = document.getElementById("adminPanel");

/* HASH */
async function hashSenha(txt){
  const buf = await crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(txt)
  );
  return Array.from(new Uint8Array(buf))
    .map(b=>b.toString(16).padStart(2,"0")).join("");
}

/* CRIA ADMIN SE NÃO EXISTIR */
const ADMIN_USER = "clx";
const ADMIN_PASS = "0110";

(async ()=>{
  const ref = doc(db,"config","admin");
  const snap = await getDoc(ref);

  if(!snap.exists()){
    await setDoc(ref,{
      usuario: ADMIN_USER,
      senha: await hashSenha(ADMIN_PASS)
    });
  }

  if(localStorage.getItem("logado")==="1"){
    loginScreen.style.display="none";
    main.style.display="block";
    adminBtn.style.display="inline";
  }
})();

/* LOGIN */
document.getElementById("btnLogin").onclick = async ()=>{
  const u = user.value.trim().toLowerCase();
  const p = await hashSenha(pass.value);

  const snap = await getDoc(doc(db,"config","admin"));
  const dados = snap.data();

  if(u===dados.usuario && p===dados.senha){
    localStorage.setItem("logado","1");
    loginScreen.style.display="none";
    main.style.display="block";
    adminBtn.style.display="inline";
  }else{
    alert("Usuário ou senha incorretos");
  }
};

/* SAIR */
document.getElementById("btnSair").onclick = ()=>{
  localStorage.clear();
  location.reload();
};

/* ADMIN */
adminBtn.onclick = ()=> adminPanel.style.display="block";
document.getElementById("fecharAdmin").onclick = ()=>{
  adminPanel.style.display="none";
};
document.getElementById("salvarSenha").onclick = async ()=>{
  const nova = novaSenha.value;
  if(!nova) return alert("Digite a senha");

  await updateDoc(doc(db,"config","admin"),{
    senha: await hashSenha(nova)
  });
  alert("Senha alterada");
  novaSenha.value="";
};
</script>

<script>
function gerar(){
  let s=+start.value,e=+end.value;
  let b=base.value==="e"?Math.E:+base.value;
  let p=+precision.value;
  let h="<table>";
  for(let i=s;i<=e;i++){
    h+=`<tr><td>${i}</td><td>${(Math.log(i)/Math.log(b)).toFixed(p)}</td></tr>`;
  }
  h+="</table>";
  preview.innerHTML=h;
}
</script>

</body>
</html>
