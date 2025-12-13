<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gerador de Tabela de Logaritmos → Excel</title>

  <style>
    :root{
      font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif
    }
    body{
      margin:0;
      background:#f3f4f6;
      color:#111
    }
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
    .row > *{flex:1}
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
    .small{font-size:13px;color:#374151}

    /* === FOOTER COM ANIMAÇÃO SUAVE === */
    footer{
      margin-top:28px;
      text-align:center;
      font-size:14px;
      color:#6b7280;
      opacity:0;
      animation: fadeUp 1.2s ease forwards;
      animation-delay:0.4s;
    }

    footer strong{
      color:#2563eb;
    }

    @keyframes fadeUp{
      from{
        opacity:0;
        transform:translateY(10px);
      }
      to{
        opacity:1;
        transform:translateY(0);
      }
    }
  </style>

  <!-- SheetJS CDN -->
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>

<body>
  <div class="container">
    <h1>Gerador de Tabela de Logaritmos — Exporta para Excel (.xlsx)</h1>
    <p class="small">
      Escolha o intervalo e a base do logaritmo; gere e baixe um arquivo Excel com a tabela pronta.
    </p>

    <div class="row">
      <div>
        <label>Início (inteiro ≥ 1)</label>
        <input id="start" type="number" value="1" min="1">
      </div>
      <div>
        <label>Fim (inteiro ≥ início)</label>
        <input id="end" type="number" value="10" min="1">
      </div>
    </div>

    <label>Base do logaritmo</label>
    <select id="base">
      <option value="10">10 (log₁₀)</option>
      <option value="e">e (ln)</option>
      <option value="2">2 (log₂)</option>
      <option value="other">Outra</option>
    </select>

    <div id="otherBaseWrap" style="display:none">
      <label>Digite a base</label>
      <input id="otherBase" type="number" step="any" value="3">
    </div>

    <label>Casas decimais (0–8)</label>
    <input id="precision" type="number" value="3" min="0" max="8">

    <div class="row">
      <button id="generate">Gerar e Mostrar Tabela</button>
      <button id="download">Gerar e Baixar Excel</button>
    </div>

    <div class="preview" id="preview"></div>

    <!-- === FOOTER COM ANO AUTOMÁTICO === -->
    <footer>
      ✨ Criado por <strong>CLX</strong> © <span id="ano"></span>
    </footer>
  </div>

  <script>
    // === ANO AUTOMÁTICO ===
    document.getElementById("ano").textContent = new Date().getFullYear();

    const baseSelect = document.getElementById('base');
    const otherWrap = document.getElementById('otherBaseWrap');

    baseSelect.onchange = ()=>{
      otherWrap.style.display = baseSelect.value === 'other' ? 'block' : 'none';
    };

    function getBase(){
      if(baseSelect.value === '10') return 10;
      if(baseSelect.value === 'e') return Math.E;
      if(baseSelect.value === '2') return 2;
      return parseFloat(document.getElementById('otherBase').value);
    }

    function buildTable(start,end,base,precision){
      const rows=[];
      for(let n=start;n<=end;n++){
        rows.push({
          n,
          log:(Math.log(n)/Math.log(base)).toFixed(precision)
        });
      }
      return rows;
    }

    function renderPreview(rows){
      let html='<table><thead><tr><th>n</th><th>log</th></tr></thead><tbody>';
      rows.forEach(r=>{
        html+=`<tr><td style="text-align:left">${r.n}</td><td>${r.log}</td></tr>`;
      });
      html+='</tbody></table>';
      document.getElementById('preview').innerHTML=html;
    }

    document.getElementById('generate').onclick=()=>{
      renderPreview(buildTable(
        +start.value,
        +end.value,
        getBase(),
        +precision.value
      ));
    };

    document.getElementById('download').onclick=()=>{
      const rows=buildTable(+start.value,+end.value,getBase(),+precision.value);
      const ws=XLSX.utils.json_to_sheet(rows);
      const wb=XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb,ws,'logaritmos');
      XLSX.writeFile(wb,'tabela_logaritmos.xlsx');
    };

    document.getElementById('generate').click();
  </script>
</body>
</html>
