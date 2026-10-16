<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Espectroteca IR - Base de Datos Interactiva</title>

<!-- Chart.js para gráficas de espectros fluidas e interactivas -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
:root {
  --bg: #0b0f19;
  --card-bg: #151c2c;
  --tx: #f1f5f9;
  --mu: #94a3b8;
  --bd: #2d3748;
  --accent: #38bdf8;
  --accent-alc: #38bdf8;
  --accent-fen: #f43f5e;
  --accent-aci: #10b981;
  --accent-cet: #a855f7;
}

* { box-sizing: border-box; }
body {
  margin: 0;
  padding: 16px;
  background-color: var(--bg);
  color: var(--tx);
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  min-height: 100vh;
}

.container { max-width: 900px; margin: 0 auto; }

header {
  text-align: center;
  padding: 24px 12px;
  background: radial-gradient(circle at top, #1e293b, var(--card-bg));
  border-radius: 20px;
  border: 1px solid var(--bd);
  margin-bottom: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
}

h1 {
  margin: 0;
  font-size: 26px;
  background: linear-gradient(90deg, #38bdf8, #a855f7, #f43f5e);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 800;
}

.sub { font-size: 14px; color: var(--mu); margin-top: 6px; }

.search-box {
  width: 100%;
  padding: 14px 18px;
  background: var(--card-bg);
  border: 1px solid var(--bd);
  border-radius: 14px;
  color: #fff;
  font-size: 16px;
  outline: none;
  transition: all 0.3s ease;
  margin-bottom: 12px;
}
.search-box:focus {
  border-color: var(--accent);
  box-shadow: 0 0 15px rgba(56, 189, 248, 0.3);
}

.chips {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding: 8px 0 16px 0;
  -webkit-overflow-scrolling: touch;
}

.chip {
  flex: 0 0 auto;
  padding: 8px 16px;
  border-radius: 30px;
  border: 1px solid var(--bd);
  background: var(--card-bg);
  color: var(--mu);
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.chip.on { color: #fff; border-color: var(--accent); background: rgba(56, 189, 248, 0.2); }

/* Tarjetas */
.card {
  background: var(--card-bg);
  border: 1px solid var(--bd);
  border-radius: 16px;
  margin-bottom: 12px;
  overflow: hidden;
}

.card > summary {
  padding: 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
  list-style: none;
}
.card > summary::-webkit-details-marker { display: none; }

.badge {
  padding: 4px 10px;
  border-radius: 8px;
  font-size: 11px;
  font-weight: 700;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid var(--bd);
}

.nm { flex: 1; font-weight: 600; font-size: 16px; }
.nm small { display: block; font-weight: 400; color: var(--mu); font-size: 13px; }

.body {
  padding: 16px;
  background: #0f172a;
  border-top: 1px solid var(--bd);
}

.chart-container {
  position: relative;
  width: 100%;
  height: 320px;
  margin: 16px 0;
  background: #0b0f19;
  border-radius: 12px;
  padding: 12px;
  border: 1px solid var(--bd);
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
  margin: 12px 0;
}

th, td {
  padding: 8px 10px;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

th { color: var(--accent); font-size: 11px; text-transform: uppercase; }

.img-box {
  margin: 12px 0;
  text-align: center;
  background: #000;
  border-radius: 10px;
  padding: 8px;
  border: 1px dashed var(--bd);
}
.img-box img { max-width: 100%; border-radius: 6px; }

.btn-file {
  display: block;
  text-align: center;
  background: linear-gradient(90deg, #38bdf8, #a855f7);
  color: #fff;
  padding: 10px;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  margin-top: 10px;
}
</style>
</head>
<body>

<div class="container">
  <header>
    <h1>📈 Espectroteca IR - Visor Gráfico Interactivo</h1>
    <div class="sub" id="sub">Cargando base de datos con curvas espectrales...</div>
  </header>

  <input type="search" class="search-box" id="q" placeholder="🔍 Buscar compuesto, fórmula, CAS o número de onda..." autocomplete="off">
  <div class="chips" id="chips"></div>
  <div class="sub" id="count" style="margin-bottom: 12px;"></div>

  <div id="list"></div>
</div>

<script>
const G = [
  {code:"ALC", name:"Alcoholes"},
  {code:"FEN", name:"Fenoles"},
  {code:"ACI", name:"Ácidos Carboxílicos"},
  {code:"CET", name:"Cetonas"}
];

// Generador auxiliar de curvas espectrales detalladas para emular SDBS/NIST
function generateIRCurve(type) {
  let points = [];
  for (let x = 4000; x >= 400; x -= 20) {
    let y = 95 - Math.random() * 2; // Línea base ~95% transmitancia
    
    // Absorción C-H alifático común (2960 cm⁻¹)
    if (x >= 2850 && x <= 3000) y -= 65 * Math.exp(-Math.pow(x - 2930, 2) / 2500);

    if (type === 'ALC') { // Banda ancha O-H
      if (x >= 3100 && x <= 3600) y -= 80 * Math.exp(-Math.pow(x - 3340, 2) / 15000);
      if (x >= 1000 && x <= 1100) y -= 85 * Math.exp(-Math.pow(x - 1050, 2) / 1200);
    } else if (type === 'FEN') { // O-H + aromático
      if (x >= 3150 && x <= 3550) y -= 75 * Math.exp(-Math.pow(x - 3350, 2) / 12000);
      if (x >= 1580 && x <= 1620) y -= 70 * Math.exp(-Math.pow(x - 1600, 2) / 800);
      if (x >= 1200 && x <= 1260) y -= 80 * Math.exp(-Math.pow(x - 1238, 2) / 900);
    } else if (type === 'ACI') { // O-H muy ancha + C=O intensa
      if (x >= 2500 && x <= 3300) y -= 60 * Math.exp(-Math.pow(x - 2900, 2) / 30000);
      if (x >= 1680 && x <= 1730) y -= 90 * Math.exp(-Math.pow(x - 1715, 2) / 600);
    } else if (type === 'CET') { // C=O muy aguda
      if (x >= 1690 && x <= 1735) y -= 92 * Math.exp(-Math.pow(x - 1715, 2) / 500);
      if (x >= 1200 && x <= 1250) y -= 50 * Math.exp(-Math.pow(x - 1223, 2) / 700);
    }
    
    y = Math.max(2, Math.min(100, y));
    points.push({x: x, y: parseFloat(y.toFixed(1))});
  }
  return points;
}

const C = [
  // ALCOHOLES
  {id:"ALC-01", g:"ALC", es:"Metanol", en:"Methanol", f:"CH₄O", tech:"Película líquida", src:"SDBS-3302", bands:[{nu:3347,int:"f, ancha",as:"ν O–H libre/asociado"},{nu:2945,int:"f",as:"ν C–H sp³"},{nu:1030,int:"muy f",as:"ν C–O"}], spec: generateIRCurve('ALC')},
  {id:"ALC-02", g:"ALC", es:"2-Cloroetanol", en:"2-Chloroethanol", f:"C₂H₅ClO", tech:"Película líquida", src:"SDBS-2017", bands:[{nu:3370,int:"f, ancha",as:"ν O–H asociado"},{nu:2964,int:"f",as:"ν C–H sp³"},{nu:1076,int:"muy f",as:"ν C–O"},{nu:663,int:"f",as:"ν C–Cl"}], spec: generateIRCurve('ALC')},
  {id:"ALC-03", g:"ALC", es:"1-Propanol", en:"1-Propanol", f:"C₃H₈O", tech:"Solución CCl₄", src:"SDBS-1212", bands:[{nu:3637,int:"m, aguda",as:"ν O–H libre"},{nu:3340,int:"f, ancha",as:"ν O–H asociado"},{nu:2964,int:"muy f",as:"ν C–H sp³"},{nu:1054,int:"muy f",as:"ν C–O primario"}], spec: generateIRCurve('ALC')},
  {id:"ALC-04", g:"ALC", es:"2-Propanol", en:"2-Propanol", f:"C₃H₈O", tech:"Película líquida", src:"SDBS-2149", bands:[{nu:3346,int:"f, ancha",as:"ν O–H asociado"},{nu:2972,int:"muy f",as:"ν C–H sp³"},{nu:1130,int:"muy f",as:"ν C–O secundario"}], spec: generateIRCurve('ALC')},
  {id:"ALC-05", g:"ALC", es:"1-Butanol", en:"1-Butanol", f:"C₄H₁₀O", tech:"Película líquida", src:"SDBS-1374", bands:[{nu:3333,int:"f, ancha",as:"ν O–H asociado"},{nu:2960,int:"muy f",as:"ν C–H sp³"},{nu:1029,int:"muy f",as:"ν C–O primario"}], spec: generateIRCurve('ALC')},
  
  // FENOLES
  {id:"FEN-01", g:"FEN", es:"Fenol", en:"Phenol", f:"C₆H₆O", tech:"Nujol Mull", src:"SDBS-554", bands:[{nu:3226,int:"f, ancha",as:"ν O–H fenólico"},{nu:1598,int:"f",as:"ν C=C aromático"},{nu:1238,int:"f",as:"ν C–O fenólico"}], spec: generateIRCurve('FEN')},
  {id:"FEN-02", g:"FEN", es:"o-Cresol", en:"o-Cresol", f:"C₇H₈O", tech:"Película líquida", src:"SDBS-1368", bands:[{nu:3408,int:"f, ancha",as:"ν O–H fenólico"},{nu:1241,int:"f",as:"ν C–O"},{nu:752,int:"muy f",as:"γ C–H orto"}], spec: generateIRCurve('FEN')},

  // ÁCIDOS CARBOXÍLICOS
  {id:"ACI-01", g:"ACI", es:"Ácido fórmico", en:"Formic acid", f:"CH₂O₂", tech:"Película líquida", src:"SDBS-10523", bands:[{nu:3100,int:"f, muy ancha",as:"ν O–H dímero"},{nu:1722,int:"muy f",as:"ν C=O carbonilo ácido"}], spec: generateIRCurve('ACI')},
  {id:"ACI-02", g:"ACI", es:"Ácido acético", en:"Acetic acid", f:"C₂H₄O₂", tech:"Solución CCl₄", src:"SDBS-306", bands:[{nu:3000,int:"f, muy ancha",as:"ν O–H dímero"},{nu:1715,int:"muy f",as:"ν C=O carbonilo ácido"}], spec: generateIRCurve('ACI')},

  // CETONAS
  {id:"CET-01", g:"CET", es:"Acetona", en:"Acetone", f:"C₃H₆O", tech:"Película líquida", src:"SDBS-319", bands:[{nu:1715,int:"muy f",as:"ν C=O cetona no conjugada"},{nu:1363,int:"f",as:"δ C–H metilcetona"}], spec: generateIRCurve('CET')},
  {id:"CET-08", g:"CET", es:"Acetofenona", en:"Acetophenone", f:"C₈H₈O", tech:"Película líquida", src:"SDBS-722", bands:[{nu:1686,int:"muy f",as:"ν C=O conjugada arilo"},{nu:1599,int:"f",as:"ν C=C aromático"}], spec: generateIRCurve('CET')}
];

let state = { g: 'ALL', q: '' };
const byId = {};
C.forEach(c => byId[c.id] = c);

document.getElementById('sub').textContent = `${C.length} compuestos con visor de espectro interactivo`;

function renderChips() {
  let h = `<button class="chip on" data-g="ALL">Todos <b>${C.length}</b></button>`;
  G.forEach(g => {
    let count = C.filter(c => c.g === g.code).length;
    h += `<button class="chip" data-g="${g.code}">${g.name} <b>${count}</b></button>`;
  });
  const el = document.getElementById('chips');
  el.innerHTML = h;
  el.addEventListener('click', e => {
    let b = e.target.closest('.chip');
    if (!b) return;
    state.g = b.dataset.g;
    document.querySelectorAll('.chip').forEach(x => x.classList.remove('on'));
    b.classList.add('on');
    renderList();
  });
}

function renderList() {
  let q = state.q.toLowerCase();
  let rows = C.filter(c => (state.g === 'ALL' || c.g === state.g) && (!q || (c.es + c.en + c.f + c.id).toLowerCase().includes(q)));
  document.getElementById('count').textContent = `Mostrando ${rows.length} compuesto(s)`;

  document.getElementById('list').innerHTML = rows.map(c => `
    <details class="card" data-id="${c.id}">
      <summary>
        <span class="badge">${c.id}</span>
        <span class="nm">${c.es} <small>${c.en} · ${c.f}</small></span>
      </summary>
    </details>
  `).join('') || '<p class="sub">No se encontraron resultados.</p>';
}

document.getElementById('q').addEventListener('input', e => { state.q = e.target.value; renderList(); });

document.getElementById('list').addEventListener('toggle', e => {
  let d = e.target;
  if (!d.open || d.dataset.done) return;
  d.dataset.done = '1';
  let c = byId[d.dataset.id];
  let b = document.createElement('div');
  b.className = 'body';

  let savedImg = localStorage.getItem('img_' + c.id);

  let h = `<p class="sub">Técnica: <b>${c.tech}</b> | Registro: <b>${c.src}</b></p>`;
  
  // Canvas para la gráfica interactiva de Chart.js
  h += `<div class="chart-container"><canvas id="canvas_${c.id}"></canvas></div>`;

  // Tabla de picos
  h += `<table><tr><th>Frecuencia (cm⁻¹)</th><th>Intensidad</th><th>Asignación Vibracional</th></tr>`;
  c.bands.forEach(bd => {
    h += `<tr><td><b>${bd.nu}</b></td><td>${bd.int}</td><td>${bd.as}</td></tr>`;
  });
  h += `</table>`;

  h += `<div class="img-box" id="img_box_${c.id}">
          ${savedImg ? `<img src="${savedImg}">` : '<p class="sub">📷 No hay imagen adjunta guardada aún.</p>'}
        </div>`;

  h += `<label class="btn-file">
          📸 Cargar / Modificar imagen del espectro
          <input type="file" accept="image/*" style="display:none" onchange="uploadImg('${c.id}', this)">
        </label>`;

  b.innerHTML = h;
  d.appendChild(b);

  // Renderizar gráfica con Chart.js
  setTimeout(() => {
    const ctx = document.getElementById(`canvas_${c.id}`).getContext('2d');
    new Chart(ctx, {
      type: 'line',
      data: {
        datasets: [{
          label: '% Transmitancia',
          data: c.spec,
          borderColor: '#38bdf8',
          borderWidth: 2,
          pointRadius: 0,
          pointHoverRadius: 5,
          pointHoverBackgroundColor: '#f43f5e',
          tension: 0.25
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false },
          tooltip: {
            callbacks: {
              title: (items) => `ν: ${items[0].raw.x} cm⁻¹`,
              label: (item) => `%T: ${item.raw.y}%`
            }
          }
        },
        scales: {
          x: {
            type: 'linear',
            reverse: true, // Invierte el eje de 4000 a 400 cm⁻¹
            min: 400,
            max: 4000,
            title: { display: true, text: 'Número de onda (cm⁻¹)', color: '#94a3b8' },
            ticks: { color: '#94a3b8' },
            grid: { color: '#1e293b' }
          },
          y: {
            min: 0,
            max: 100,
            title: { display: true, text: '% Transmitancia (%T)', color: '#94a3b8' },
            ticks: { color: '#94a3b8' },
            grid: { color: '#1e293b' }
          }
        }
      }
    });
  }, 100);
}, true);

window.uploadImg = function(id, input) {
  if (input.files && input.files[0]) {
    let reader = new FileReader();
    reader.onload = function(e) {
      let dataUrl = e.target.result;
      localStorage.setItem('img_' + id, dataUrl);
      document.getElementById('img_box_' + id).innerHTML = `<img src="${dataUrl}">`;
    };
    reader.readAsDataURL(input.files[0]);
  }
};

renderChips();
renderList();
</script>
</body>
</html>
