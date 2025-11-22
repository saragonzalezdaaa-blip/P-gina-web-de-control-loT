<!-- Guarda esto como: simulador_cadena_frio.html -->
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Simulador - Monitoreo Cadena de Frío</title>
  <style>
    :root{
      --blue:#0b63b3;
      --blue-600:#0a58a5;
      --bg:#f6f8fb;
      --card:#ffffff;
      --danger:#e74c3c;
      --ok:#27ae60;
      --muted:#6b7280;
      font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    html,body{height:100%;margin:0;background:var(--bg);color:#0b1220}
    .container{max-width:980px;margin:28px auto;padding:18px;}
    header{display:flex;align-items:center;gap:16px;margin-bottom:18px}
    .brand{display:flex;flex-direction:column}
    h1{margin:0;font-size:20px;color:var(--blue-600)}
    p.lead{margin:0;color:var(--muted);font-size:13px}
    .grid{display:grid;grid-template-columns:350px 1fr;gap:16px}
    .card{background:var(--card);border-radius:12px;padding:14px;box-shadow:0 6px 18px rgba(11,19,32,0.06)}
    .big-read{display:flex;align-items:center;gap:12px}
    .metric{font-weight:700;color:var(--blue-600}
    .temp-value{font-size:48px;font-weight:800;color:#0b1220}
    .hum-value{font-size:36px;color:#0b1220}
    .small{font-size:13px;color:var(--muted)}
    .row{display:flex;gap:12px;align-items:center}
    .btn{background:var(--blue);color:white;padding:8px 12px;border-radius:8px;border:0;cursor:pointer;font-weight:600}
    .btn.secondary{background:#eef4fb;color:var(--blue-600;border:1px solid #d6e7fb}
    .btn.danger{background:var(--danger)}
    canvas{background:linear-gradient(180deg,#fafcff,#f2f7ff);border-radius:8px}
    .controls{display:flex;flex-direction:column;gap:8px;margin-top:12px}
    .label{font-size:12px;color:var(--muted)}
    .alert{display:flex;align-items:center;gap:10px;background:var(--danger);color:white;padding:10px;border-radius:8px;font-weight:700}
    .okbox{display:flex;align-items:center;gap:10px;background:var(--ok);color:white;padding:8px;border-radius:8px;font-weight:700}
    footer{margin-top:16px;font-size:12px;color:var(--muted);text-align:center}
    .log{max-height:200px;overflow:auto;padding:8px;border-radius:8px;background:#fbfdff;border:1px solid #eef5ff;font-size:13px}
    label.switch{display:inline-flex;cursor:pointer;align-items:center;gap:8px}
    input[type=range]{width:100%}
    .top-metrics{display:flex;flex-direction:column;gap:10px}
    .metric-card{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:8px;background:#fbfdff;border:1px solid #eef5ff}
    @media(max-width:880px){
      .grid{grid-template-columns:1fr; }
      .metric-card{flex-direction:column;align-items:flex-start;gap:8px}
    }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <div style="width:52px;height:52px;border-radius:10px;background:linear-gradient(135deg,var(--blue),var(--blue-600));display:flex;align-items:center;justify-content:center;color:white;font-weight:800">CF</div>
      <div class="brand">
        <h1>Simulador — Monitoreo Cadena de Frío</h1>
        <p class="lead">Prototipo web: visualización en tiempo real, alertas y log. Temp límite: 8 °C</p>
      </div>
    </header>

    <main class="grid">
      <!-- columna izquierda: métricas y controles -->
      <div class="card">
        <div class="top-metrics">
          <div class="metric-card">
            <div>
              <div class="small">Temperatura</div>
              <div class="temp-value" id="tempValue">4.5 °C</div>
            </div>
            <div style="text-align:right">
              <div class="small">Humedad</div>
              <div class="hum-value" id="humValue">65 %</div>
            </div>
          </div>

          <div class="metric-card">
            <div>
              <div class="small">Estado</div>
              <div id="statusBox" class="okbox" style="background:var(--ok)">NORMAL</div>
            </div>
            <div style="text-align:right">
              <div class="small">Última lectura</div>
              <div id="lastTime" class="small">—</div>
            </div>
          </div>
        </div>

        <hr style="border:none;height:12px">

        <div style="display:flex;gap:8px;">
          <button class="btn" id="btnStart">Iniciar simulación</button>
          <button class="btn secondary" id="btnStop">Detener</button>
          <button class="btn secondary" id="btnCSV">Descargar CSV</button>
        </div>

        <div class="controls">
          <div>
            <div class="label">Simular temperatura manual</div>
            <input id="tempSlider" type="range" min="0" max="20" step="0.1" value="4"/>
            <div style="display:flex;justify-content:space-between"><small>0 °C</small><small>20 °C</small></div>
          </div>

          <div style="display:flex;gap:8px;">
            <label class="switch">
              <input id="autoMode" type="checkbox" /> <span class="small">Modo automático (variación aleatoria)</span>
            </label>
          </div>

          <div style="display:flex;gap:8px;">
            <button class="btn secondary" id="btnSpike">Simular pico (9 °C)</button>
            <button class="btn secondary" id="btnReset">Reset historial</button>
          </div>
        </div>

        <hr>

        <div>
          <div class="label">Log de eventos</div>
          <div class="log" id="logBox">Sistema listo — sin lecturas.</div>
        </div>
      </div>

      <!-- columna derecha: gráfico y alertas -->
      <div>
        <div class="card" style="margin-bottom:14px">
          <div style="display:flex;justify-content:space-between;align-items:center;">
            <div>
              <div class="small">Histórico (últimos 60 valores)</div>
              <div style="font-weight:700">Gráfica de temperatura y humedad</div>
            </div>
            <div style="text-align:right">
              <div class="small">Umbral temp</div>
              <div style="font-weight:700">8 °C</div>
            </div>
          </div>
          <div style="height:320px;margin-top:12px">
            <canvas id="chart" width="800" height="320"></canvas>
          </div>
        </div>

        <div class="card" id="alertArea" style="display:none;margin-bottom:12px">
          <div class="alert" id="alertText">¡ALERTA! Temperatura fuera de rango: 9.2 °C</div>
        </div>

        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
            <div style="font-weight:700">Detalles de la última lectura</div>
            <div class="small">Sensor: DHT22 · Nodo: ESP32 · Local: Cámara 1</div>
          </div>
          <div style="display:flex;gap:12px">
            <div style="flex:1">
              <div class="small">Temperatura</div>
              <div id="detailTemp" style="font-size:28px;font-weight:800">4.5 °C</div>
            </div>
            <div style="flex:1">
              <div class="small">Humedad</div>
              <div id="detailHum" style="font-size:28px;font-weight:800">65 %</div>
            </div>
            <div style="flex:1">
              <div class="small">Acciones</div>
              <div style="display:flex;gap:8px;margin-top:6px">
                <button class="btn secondary" id="btnAjust">Ajustar refrigeración</button>
                <button class="btn secondary" id="btnMove">Trasladar producto</button>
              </div>
            </div>
          </div>
        </div>

      </div>
    </main>

    <footer>
      Simulador creado para evidenciar la Etapa 4 — Proyecto cadena de frío. Puede usarse en presentaciones y anexos.
    </footer>
  </div>


  <script>
    // ====== Config / Estado ======
    const LIMIT_TEMP = 8.0;
    let simInterval = null;
    let autoMode = false;
    let history = []; // {t, temp, hum, time}
    const MAX_HISTORY = 60;

    // DOM
    const tempValue = document.getElementById('tempValue');
    const humValue = document.getElementById('humValue');
    const statusBox = document.getElementById('statusBox');
    const lastTime = document.getElementById('lastTime');
    const logBox = document.getElementById('logBox');
    const chart = document.getElementById('chart');
    const alertArea = document.getElementById('alertArea');
    const alertText = document.getElementById('alertText');
    const detailTemp = document.getElementById('detailTemp');
    const detailHum = document.getElementById('detailHum');

    // controls
    const btnStart = document.getElementById('btnStart');
    const btnStop = document.getElementById('btnStop');
    const btnSpike = document.getElementById('btnSpike');
    const btnCSV = document.getElementById('btnCSV');
    const btnReset = document.getElementById('btnReset');
    const tempSlider = document.getElementById('tempSlider');
    const autoCheckbox = document.getElementById('autoMode');
    const btnAjust = document.getElementById('btnAjust');
    const btnMove = document.getElementById('btnMove');

    // initial values
    let currentTemp = parseFloat(tempSlider.value);
    let currentHum = 60 + Math.round(Math.random()*15);
    updateDisplays();

    // ====== Chart simple drawing (Canvas) ======
    const ctx = chart.getContext('2d');
    function drawChart(){
      const w = chart.width;
      const h = chart.height;
      ctx.clearRect(0,0,w,h);

      // background grid
      ctx.fillStyle = '#ffffff';
      ctx.fillRect(0,0,w,h);
      ctx.strokeStyle = '#f0f6ff';
      for(let i=0;i<6;i++){
        const y = 20 + i*(h-40)/5;
        ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(w,y);ctx.stroke();
      }

      // axes labels
      ctx.fillStyle = '#102030';
      ctx.font = '12px Inter, Arial';
      ctx.fillText('Tiempo →', w-70, h-8);

      // prepare data arrays
      const temps = history.map(x=>x.temp);
      const hums = history.map(x=>x.hum);
      const n = Math.max(1, history.length);
      // scales: y from -5 to 25 C (visual)
      const ymin = -5, ymax = 25;
      const yscale = (h-40)/(ymax-ymin);
      const xstep = (w-60)/(Math.max(1,n-1));

      // draw humidity (dashed line)
      if(hums.length>0){
        ctx.beginPath();
        for(let i=0;i<hums.length;i++){
          const x = 40 + i*xstep;
          const y = 20 + (ymax - hums[i])*yscale;
          if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        }
        ctx.strokeStyle = '#b3d4ff';
        ctx.lineWidth = 2;
        ctx.stroke();
        // label
        ctx.fillStyle = '#6b7280';
        ctx.fillText('Humedad (%)', 45, 18);
      }

      // draw temp (solid line)
      if(temps.length>0){
        ctx.beginPath();
        for(let i=0;i<temps.length;i++){
          const x = 40 + i*xstep;
          const y = 20 + (ymax - temps[i])*yscale;
          if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        }
        ctx.strokeStyle = '#0b63b3';
        ctx.lineWidth = 3;
        ctx.stroke();
        ctx.fillStyle = '#0b63b3';
        ctx.fillText('Temperatura (°C)', 160, 18);
      }

      // draw limit line
      const ylimit = 20 + (ymax - LIMIT_TEMP)*yscale;
      ctx.beginPath();ctx.moveTo(40,ylimit);ctx.lineTo(w-10,ylimit);
      ctx.strokeStyle = 'rgba(231,76,60,0.8)';ctx.setLineDash([6,6]);ctx.lineWidth=1.5;ctx.stroke();
      ctx.setLineDash([]);
      ctx.fillStyle = 'rgba(231,76,60,0.9)';
      ctx.fillText('Umbral ' + LIMIT_TEMP + '°C', w-130, ylimit - 6);
    }

    // ====== Helpers ======
    function nowStr(){ return new Date().toLocaleTimeString(); }
    function pushHistory(temp,hum){
      history.push({temp:parseFloat(temp), hum:parseFloat(hum), time: new Date()});
      if(history.length>MAX_HISTORY) history.shift();
      drawChart();
    }
    function log(msg){
      const line = `[${nowStr()}] ${msg}`;
      logBox.innerText = line + '\n' + logBox.innerText;
    }
    function updateDisplays(){
      tempValue.innerText = currentTemp.toFixed(1) + ' °C';
      humValue.innerText = currentHum.toFixed(0) + ' %';
      detailTemp.innerText = currentTemp.toFixed(1) + ' °C';
      detailHum.innerText = currentHum.toFixed(0) + ' %';
      lastTime.innerText = history.length ? history[history.length-1].time.toLocaleTimeString() : '—';
      // status
      if(currentTemp > LIMIT_TEMP){
        statusBox.innerText = 'FUERA DE RANGO';
        statusBox.style.background = 'var(--danger)';
        statusBox.style.color = 'white';
        alertArea.style.display = 'block';
        alertText.innerText = `¡ALERTA! Temperatura fuera de rango: ${currentTemp.toFixed(1)} °C`;
      } else {
        statusBox.innerText = 'NORMAL';
        statusBox.style.background = 'var(--ok)';
        statusBox.style.color = 'white';
        alertArea.style.display = 'none';
      }
    }

    // ====== Simulation loop ======
    function stepSimulation(){
      // if auto mode: add small random walk
      if(autoMode){
        const dt = (Math.random()-0.45)*0.4; // small drift
        currentTemp = Math.max(-5, Math.min(25, currentTemp + dt));
        // humidity inverse correlated a poco
        currentHum = Math.max(20, Math.min(95, currentHum + (Math.random()-0.5)*1.5));
      } else {
        // follow slider
        currentTemp = parseFloat(tempSlider.value);
      }
      pushHistory(currentTemp, currentHum);
      updateDisplays();

      // log alerts if needed
      if(currentTemp > LIMIT_TEMP){
        log(`ALERTA: Temp ${currentTemp.toFixed(1)} °C > ${LIMIT_TEMP} °C`);
      } else {
        log(`Lectura: Temp ${currentTemp.toFixed(1)} °C · Hum ${currentHum.toFixed(0)} %`);
      }
    }

    function startSim(){
      if(simInterval) return;
      simInterval = setInterval(stepSimulation, 1500); // cada 1.5s
      log('Simulación iniciada');
    }
    function stopSim(){
      if(simInterval) clearInterval(simInterval);
      simInterval = null;
      log('Simulación detenida');
    }

    // ====== Events ======
    btnStart.addEventListener('click', ()=>{ startSim(); });
    btnStop.addEventListener('click', ()=>{ stopSim(); });
    tempSlider.addEventListener('input', ()=>{ if(!autoMode){ currentTemp = parseFloat(tempSlider.value); updateDisplays(); }});
    autoCheckbox.addEventListener('change', (e)=>{ autoMode = e.target.checked; log('Modo automático: ' + (autoMode ? 'ON' : 'OFF')); });
    btnSpike.addEventListener('click', ()=>{
      currentTemp = 9.2; currentHum = currentHum + 2;
      pushHistory(currentTemp,currentHum); updateDisplays();
      log('Simulado pico de temperatura a 9.2 °C');
    });
    btnReset.addEventListener('click', ()=>{
      history = []; drawChart(); log('Historial reseteado');
    });

    btnAjust.addEventListener('click', ()=>{ log('Acción: ajustar refrigeración (simulado)'); alert('Acción: ajuste de refrigeración enviado (simulado)');});
    btnMove.addEventListener('click', ()=>{ log('Acción: trasladar producto (simulado)'); alert('Acción: traslado enviado (simulado)');});

    // CSV export
    btnCSV.addEventListener('click', ()=>{
      if(history.length===0){ alert('No hay datos para exportar'); return; }
      let csv = 'time,temp_C,hum_percent\n';
      history.forEach(r=>{ csv += `${r.time.toISOString()},${r.temp.toFixed(2)},${r.hum.toFixed(1)}\n`; });
      const blob = new Blob([csv],{type:'text/csv;charset=utf-8;'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'historial_cadena_frio.csv'; a.click();
      URL.revokeObjectURL(url);
      log('Historial exportado en CSV');
    });

    // start small initial history
    for(let i=0;i<6;i++){
      const t = 4 + Math.random()*0.8;
      const h = 60 + Math.random()*6;
      history.push({temp:t, hum:h, time: new Date(Date.now() - (6-i)*1500)});
    }
    drawChart();

    // ensure chart redraw on resize
    window.addEventListener('resize', ()=> drawChart());

    // auto start when page opens
    // startSim();   // comentado: deja que el usuario inicie
  </script>
</body>
</html><img width="1024" height="1536" alt="1000207305" src="https://github.com/user-attachments/assets/4e734375-df97-4a7e-a1a2-9e34168a2910" />
