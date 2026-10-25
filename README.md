<!DOCTYPE html>
<html lang="cs" class="h-full bg-black">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>SEMAFORY LIBEREC</title>
  
  <meta name="theme-color" content="#000000">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Segoe UI", Roboto, sans-serif;
      background-color: #000000;
      color: #ffffff;
      overflow: hidden;
    }
    #map {
      height: 100vh;
      width: 100vw;
      background: #000000;
      z-index: 1;
    }
    .leaflet-control-attribution, .leaflet-control-zoom {
      display: none !important;
    }

    /* PŮVODNÍ VELKÁ SVÍTÍCÍ TLAČÍTKA NA ÚVODNÍ STRÁNCE */
    .initial-circle-btn {
      width: 110px;
      height: 110px;
      border-radius: 50%;
      background: #000000;
      border: 4px solid #ffffff;
      box-shadow: 0 0 20px rgba(255, 255, 255, 0.8), inset 0 0 10px rgba(255, 255, 255, 0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: transform 0.15s ease, box-shadow 0.15s ease;
    }
    .initial-circle-btn:active {
      transform: scale(0.92);
      box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
    }

    /* KULATÉ DYNAMICKÉ IKONY NA MAPĚ */
    .map-traffic-icon {
      background: #000000;
      border: 2px solid #ffffff;
      border-radius: 50%;
      width: 44px !important;
      height: 44px !important;
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: 0 0 15px rgba(255, 255, 255, 0.4), 0 8px 20px rgba(0,0,0,0.9);
      cursor: pointer;
    }

    .pulse-green {
      box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7);
      animation: pulseGreen 2s infinite;
    }
    @keyframes pulseGreen {
      0% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7); }
      70% { box-shadow: 0 0 0 10px rgba(34, 197, 94, 0); }
      100% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); }
    }
  </style>
</head>
<body class="h-full w-full bg-black text-white antialiased select-none">

  <!-- ÚVODNÍ OBRAZOVKA PŘESNĚ PODLE PRVNÍHO SCREENSHOTU -->
  <div id="roleModal" class="fixed inset-0 z-50 bg-black flex flex-col items-center justify-between py-12 px-6">
    <div class="w-full"></div>
    
    <div class="text-center w-full max-w-md flex flex-col items-center">
      <h1 class="text-2xl font-bold tracking-tight text-white mb-12">Vybele typ dopravy:</h1>
      
      <!-- Dvě velká černá tlačítka s bílým zářícím kroužkem vedle sebe -->
      <div class="flex items-center justify-center space-x-10">
        <!-- Chodec -->
        <button onclick="selectInitialRole('pedestrian')" class="initial-circle-btn">
          <svg class="w-16 h-16 text-white" fill="currentColor" viewBox="0 0 24 24">
            <path d="M13.5 5.5c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zM9.8 8.9L7 23h2.1l1.8-8 2.1 2v6h2v-7.5l-2.1-2 .6-3c1.3 1.5 3.3 2.5 5.5 2.5v-2c-1.9 0-3.5-1-4.3-2.4l-1-1.6c-.4-.6-1-1-1.7-1-.3 0-.5.1-.8.1L7 9.5V14h2V9.9l.8-1z"/>
          </svg>
        </button>

        <!-- Auto -->
        <button onclick="selectInitialRole('car')" class="initial-circle-btn">
          <svg class="w-16 h-16 text-white" fill="currentColor" viewBox="0 0 24 24">
            <path d="M18.92 6.01C18.72 5.42 18.16 5 17.5 5h-11c-.66 0-1.22.42-1.42 1.01L3 12v8c0 .55.45 1 1 1h1c.55 0 1-.45 1-1v-1h12v1c0 .55.45 1 1 1h1c.55 0 1-.45 1-1v-8l-2.08-5.99zM6.85 7h10.29l1.04 3H5.81l1.04-3zM19 17H5v-4.66l.12-.34h13.76l.12.34V17z"/>
            <circle cx="7.5" cy="14.5" r="1.5"/>
            <circle cx="16.5" cy="14.5" r="1.5"/>
          </svg>
        </button>
      </div>
    </div>

    <!-- GPS potvrzení na spodku obrazovky -->
    <div class="bg-neutral-900/90 border border-neutral-800 rounded-lg px-4 py-2 text-xs text-neutral-300 font-mono flex items-center space-x-2">
      <span class="text-green-500 font-bold">✓</span>
      <span>Povelit pffs6 p > pefera GPS</span>
    </div>
  </div>

  <!-- HLAVNÍ MAPA -->
  <main class="relative h-full w-full">
    <div id="map"></div>

    <!-- Tlačítko přepínání role vpravo nahoře -->
    <button onclick="toggleRole()" class="fixed top-5 right-5 z-30 w-14 h-14 bg-black/90 border-2 border-white/80 rounded-full flex items-center justify-center shadow-[0_0_15px_rgba(255,255,255,0.4)] backdrop-blur-xl active:scale-95 transition">
      <span id="topRoleIcon" class="text-2xl">🚶</span>
    </button>

    <!-- Apple Maps styl spodní lišta -->
    <div class="fixed bottom-6 left-4 right-4 z-30 max-w-md mx-auto bg-neutral-900/90 border border-neutral-800/80 rounded-full px-5 py-3.5 shadow-2xl backdrop-blur-2xl flex items-center justify-between">
      <div class="flex items-center space-x-3">
        <svg class="w-5 h-5 text-neutral-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
        </svg>
        <span id="barSignalName" class="text-sm font-semibold tracking-tight text-white">Šaldovo náměstí</span>
      </div>
      <div class="flex items-center space-x-3">
        <button onclick="openCountdownOverlay()" class="bg-white text-black font-black text-[11px] uppercase px-4 py-2 rounded-full transition active:scale-95">
          ODPOČET
        </button>
        <div class="w-8 h-8 rounded-full bg-neutral-800 border border-neutral-700 flex items-center justify-center text-xs font-mono font-bold text-neutral-300">
          MH
        </div>
      </div>
    </div>
  </main>

  <!-- FULLSCREEN COUNTDOWN OVERLAY -->
  <div id="countdownOverlay" class="fixed inset-0 z-50 bg-black flex flex-col justify-between p-6 hidden transition-opacity duration-300">
    <div class="flex items-center justify-between">
      <button onclick="closeCountdownOverlay()" class="text-neutral-400 hover:text-white p-2 border border-neutral-800 rounded-full bg-neutral-950">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
        </svg>
      </button>
      <div class="text-center">
        <p id="overlaySignalName" class="text-xs font-mono font-bold uppercase tracking-widest text-neutral-400">ŠALDOVO NÁMĚSTÍ</p>
        <p id="overlayStatus" class="text-[11px] font-mono text-red-500 uppercase tracking-wider mt-0.5">SEKUND DO KONCE ČERVENÉ</p>
      </div>
      <div class="w-10"></div>
    </div>

    <div class="flex flex-col items-center justify-center my-auto">
      <span id="timerValue" class="text-[130px] font-black leading-none tracking-tighter text-red-500 font-mono">18</span>
      <div id="signalIconBox" class="mt-4 flex justify-center items-center">
        <!-- CHODEC SVG -->
        <svg id="countdownPedSvg" class="w-24 h-24 transition-colors duration-300" fill="currentColor" viewBox="0 0 24 24" style="fill: #ef4444;">
          <path d="M13.5 5.5c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zM9.8 8.9L7 23h2.1l1.8-8 2.1 2v6h2v-7.5l-2.1-2 .6-3c1.3 1.5 3.3 2.5 5.5 2.5v-2c-1.9 0-3.5-1-4.3-2.4l-1-1.6c-.4-.6-1-1-1.7-1-.3 0-.5.1-.8.1L7 9.5V14h2V9.9l.8-1z"/>
        </svg>
        <!-- AUTO SEMAFOR MODUL -->
        <div id="countdownCarLight" class="hidden flex-col items-center bg-neutral-900 border-2 border-neutral-800 p-3 rounded-2xl space-y-2.5">
          <div id="carRedDot" class="w-8 h-8 rounded-full bg-red-500 shadow-[0_0_15px_rgba(239,68,68,0.9)]"></div>
          <div id="carYellowDot" class="w-8 h-8 rounded-full bg-neutral-800"></div>
          <div id="carGreenDot" class="w-8 h-8 rounded-full bg-neutral-800"></div>
        </div>
      </div>
    </div>

    <div class="border-t border-neutral-900 pt-4 flex items-center justify-between text-xs text-neutral-500 font-mono">
      <span id="cycleInfoOverlay">CYKLUS: RED (34s) / GREEN (22s)</span>
      <span class="text-green-500 flex items-center gap-1">
        <span class="w-2 h-2 rounded-full bg-green-500 pulse-green"></span> LIVE SYNC OK
      </span>
    </div>
  </div>

  <script>
    const STADIA_API_KEY = 'cb6e4259-4763-41dd-a6b3-1efc1e70f6f0';
    
    // VŠECHNY BODY V LIBERCI PŘÍSTUPNÉ PRO OBĚ ROLE
    const baseSignals = [
      { id: "node/73315982", name: "Sokolská", lat: 50.7641882, lng: 15.0713986, redTime: 30, greenTime: 20 },
      { id: "node/73343363", name: "Sokolská / Jablonecká", lat: 50.7700068, lng: 15.0612162, redTime: 34, greenTime: 22 },
      { id: "node/73344025", name: "Palachova křižovatka", lat: 50.765892, lng: 15.0675612, redTime: 40, greenTime: 25 },
      { id: "node/73357840", name: "Růžodol okruh", lat: 50.773874, lng: 15.039904, redTime: 35, greenTime: 20 },
      { id: "node/73375642", name: "Gorkého křižovatka", lat: 50.7655646, lng: 15.0684031, redTime: 30, greenTime: 20 },
      { id: "node/73375693", name: "Jablonecká výjezd", lat: 50.7621663, lng: 15.0755887, redTime: 32, greenTime: 18 },
      { id: "node/73380990", name: "Rochlice spojka", lat: 50.7429214, lng: 15.096239, redTime: 45, greenTime: 30 },
      { id: "node/73382418", name: "Vratislavice hlavní", lat: 50.7454327, lng: 15.0858183, redTime: 35, greenTime: 20 },
      { id: "node/73384479", name: "Fügnerova přechod", lat: 50.768061, lng: 15.065269, redTime: 25, greenTime: 15 },
      { id: "node/73413042", name: "Sokolská u pošty", lat: 50.7701231, lng: 15.0605862, redTime: 34, greenTime: 22 },
      { id: "node/204942172", name: "Šaldovo náměstí - centrální", lat: 50.7652756, lng: 15.0583574, redTime: 40, greenTime: 25 },
      { id: "node/206218158", name: "Chrastavská", lat: 50.770308, lng: 15.0486791, redTime: 30, greenTime: 15 },
      { id: "node/206219561", name: "Husova / Zeyerova", lat: 50.7580766, lng: 15.0500974, redTime: 30, greenTime: 20 },
      { id: "node/247991725", name: "Průmyslová zóna Jih", lat: 50.7606152, lng: 15.0898927, redTime: 50, greenTime: 20 },
      { id: "node/247992230", name: "Karloveská", lat: 50.765691, lng: 15.0437212, redTime: 30, greenTime: 20 },
      { id: "node/248466920", name: "Doubí", lat: 50.74438, lng: 15.048164, redTime: 35, greenTime: 20 },
      { id: "node/249079815", name: "Ruprechtická", lat: 50.7823041, lng: 15.0414042, redTime: 30, greenTime: 18 },
      { id: "node/252750060", name: "Partyzánská", lat: 50.7663805, lng: 15.0444593, redTime: 30, greenTime: 20 },
      { id: "node/254096099", name: "Ostašov", lat: 50.7508794, lng: 15.0323568, redTime: 30, greenTime: 20 },
      { id: "node/280900049", name: "Lidových sadů", lat: 50.7576487, lng: 15.0595034, redTime: 30, greenTime: 20 },
      { id: "node/280900052", name: "Zhořelecká", lat: 50.7556142, lng: 15.0553644, redTime: 30, greenTime: 20 },
      { id: "node/280900060", name: "Rochlická", lat: 50.7482348, lng: 15.0629583, redTime: 35, greenTime: 22 },
      { id: "node/280987496", name: "Jindřichovská", lat: 50.7432138, lng: 15.0602644, redTime: 30, greenTime: 20 },
      { id: "node/309418901", name: "Pilínkov", lat: 50.7386936, lng: 15.0140781, redTime: 30, greenTime: 20 },
      { id: "node/335648201", name: "Machnín", lat: 50.7324613, lng: 14.984385, redTime: 40, greenTime: 20 },
      { id: "node/501666540", name: "Šaldovo náměstí - přechod", lat: 50.7697504, lng: 15.0578057, redTime: 34, greenTime: 22 },
      { id: "node/604466234", name: "Střelecký vrch", lat: 50.7647285, lng: 15.0547898, redTime: 30, greenTime: 20 },
      { id: "node/606095932", name: "U Nisy", lat: 50.7642567, lng: 15.0640208, redTime: 30, greenTime: 20 },
      { id: "node/612457980", name: "Pražská 1", lat: 50.7685074, lng: 15.0601612, redTime: 30, greenTime: 20 },
      { id: "node/612457983", name: "Pražská 2", lat: 50.7683891, lng: 15.0603504, redTime: 30, greenTime: 20 },
      { id: "node/612457992", name: "Sokolská přechod", lat: 50.7699282, lng: 15.060669, redTime: 34, greenTime: 22 },
      { id: "node/1376124389", name: "Měnírna", lat: 50.7531051, lng: 15.0557316, redTime: 30, greenTime: 20 },
      { id: "node/1499469228", name: "U Nisy 2", lat: 50.7642619, lng: 15.0636674, redTime: 25, greenTime: 15 },
      { id: "node/1667412278", name: "Harcov přehrada", lat: 50.7808303, lng: 15.0395824, redTime: 30, greenTime: 20 },
      { id: "node/1667422226", name: "Harcov", lat: 50.7806778, lng: 15.0399955, redTime: 30, greenTime: 15 },
      { id: "node/1989835250", name: "Masarykova", lat: 50.7607593, lng: 15.054464, redTime: 35, greenTime: 22 },
      { id: "node/2919270501", name: "Ještědská", lat: 50.7393462, lng: 15.0172514, redTime: 30, greenTime: 15 },
      { id: "node/5194272549", name: "Karloveská 2", lat: 50.7655952, lng: 15.0433622, redTime: 30, greenTime: 20 },
      { id: "node/5305456904", name: "Rumunská", lat: 50.7644889, lng: 15.0556387, redTime: 30, greenTime: 20 }
    ];

    let activeMarkers = [];
    let userLat = 50.7671;
    let userLng = 15.0572;
    let userRole = 'pedestrian';
    let activeSignal = baseSignals[0];

    const map = L.map('map', { zoomControl: false }).setView([userLat, userLng], 14);

    L.tileLayer(`https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png?api_key=${STADIA_API_KEY}`, {
      maxZoom: 20
    }).addTo(map);

    const userMarker = L.circleMarker([userLat, userLng], {
      color: '#22c55e',
      fillColor: '#22c55e',
      fillOpacity: 0.9,
      radius: 8
    }).addTo(map);

    function selectInitialRole(role) {
      userRole = role;
      document.getElementById('roleModal').classList.add('hidden');
      updateRoleDisplay();
      renderMapMarkers();
    }

    function toggleRole() {
      userRole = userRole === 'pedestrian' ? 'car' : 'pedestrian';
      updateRoleDisplay();
      renderMapMarkers();
    }

    function updateRoleDisplay() {
      const isPed = userRole === 'pedestrian';
      document.getElementById('topRoleIcon').innerText = isPed ? '🚶' : '🚗';
      
      const pedSvg = document.getElementById('countdownPedSvg');
      const carLight = document.getElementById('countdownCarLight');

      if (isPed) {
        pedSvg.classList.remove('hidden');
        carLight.classList.add('hidden');
        carLight.classList.remove('flex');
      } else {
        pedSvg.classList.add('hidden');
        carLight.classList.remove('hidden');
        carLight.classList.add('flex');
      }
    }

    // VYGENEROVÁNÍ NÁHLEDŮ IKON PRO MAPU
    function getMarkerIconHtml(signalState) {
      if (userRole === 'car') {
        // 3-barevný vertikální semafor pro auta
        const isRed = signalState === 'RED';
        return `
          <div class="map-traffic-icon">
            <div class="flex flex-col items-center bg-neutral-900 border border-neutral-700 p-1 rounded-full space-y-1">
              <div class="w-2.5 h-2.5 rounded-full ${isRed ? 'bg-red-500 shadow-[0_0_8px_rgba(239,68,68,1)]' : 'bg-neutral-800'}"></div>
              <div class="w-2.5 h-2.5 rounded-full bg-neutral-800"></div>
              <div class="w-2.5 h-2.5 rounded-full ${!isRed ? 'bg-green-500 shadow-[0_0_8px_rgba(34,197,94,1)]' : 'bg-neutral-800'}"></div>
            </div>
          </div>
        `;
      } else {
        // 2-barevný semafor pro chodce se svítícím panáčkem
        const isRed = signalState === 'RED';
        const pedColor = isRed ? '#ef4444' : '#22c55e';
        return `
          <div class="map-traffic-icon">
            <svg class="w-6 h-6" viewBox="0 0 24 24" fill="${pedColor}">
              <path d="M13.5 5.5c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zM9.8 8.9L7 23h2.1l1.8-8 2.1 2v6h2v-7.5l-2.1-2 .6-3c1.3 1.5 3.3 2.5 5.5 2.5v-2c-1.9 0-3.5-1-4.3-2.4l-1-1.6c-.4-.6-1-1-1.7-1-.3 0-.5.1-.8.1L7 9.5V14h2V9.9l.8-1z"/>
            </svg>
          </div>
        `;
      }
    }

    function renderMapMarkers() {
      activeMarkers.forEach(m => map.removeLayer(m.marker));
      activeMarkers = [];

      const nowSec = Math.floor(Date.now() / 1000);

      baseSignals.forEach(signal => {
        const totalCycle = signal.redTime + signal.greenTime;
        const currentInCycle = nowSec % totalCycle;
        const signalState = currentInCycle < signal.redTime ? 'RED' : 'GREEN';

        const customIcon = L.divIcon({
          className: '',
          html: getMarkerIconHtml(signalState),
          iconSize: [44, 44],
          iconAnchor: [22, 22]
        });

        const marker = L.marker([signal.lat, signal.lng], { icon: customIcon }).addTo(map);
        marker.on('click', () => {
          selectSignal(signal);
          openCountdownOverlay();
        });

        activeMarkers.push({ id: signal.id, marker: marker, signal: signal });
      });

      if (baseSignals.length > 0) {
        selectSignal(baseSignals[0]);
      }
    }

    function selectSignal(signal) {
      activeSignal = signal;
      document.getElementById('barSignalName').innerText = signal.name;
      document.getElementById('overlaySignalName').innerText = signal.name;
    }

    function openCountdownOverlay() {
      document.getElementById('countdownOverlay').classList.remove('hidden');
    }

    function closeCountdownOverlay() {
      document.getElementById('countdownOverlay').classList.add('hidden');
    }

    // TRVALÝ ČASOVAČ BĚŽÍCÍ NA POZADÍ
    setInterval(() => {
      const nowSec = Math.floor(Date.now() / 1000);

      // Aktualizace ikonek na mape
      activeMarkers.forEach(item => {
        const totalCycle = item.signal.redTime + item.signal.greenTime;
        const currentInCycle = nowSec % totalCycle;
        const signalState = currentInCycle < item.signal.redTime ? 'RED' : 'GREEN';
        
        const el = item.marker.getElement();
        if (el) {
          const iconBox = el.querySelector('.map-traffic-icon');
          if (iconBox) {
            iconBox.parentElement.innerHTML = getMarkerIconHtml(signalState);
          }
        }
      });

      // Aktualizace vybraného odpočtu
      const totalCycle = activeSignal.redTime + activeSignal.greenTime;
      const currentInCycle = nowSec % totalCycle;

      let state = 'RED';
      let remaining = 0;

      if (currentInCycle < activeSignal.redTime) {
        state = 'RED';
        remaining = activeSignal.redTime - currentInCycle;
      } else {
        state = 'GREEN';
        remaining = totalCycle - currentInCycle;
      }

      const timerVal = document.getElementById('timerValue');
      const statusText = document.getElementById('overlayStatus');
      const cycleInfo = document.getElementById('cycleInfoOverlay');
      const pedSvg = document.getElementById('countdownPedSvg');
      const carRedDot = document.getElementById('carRedDot');
      const carGreenDot = document.getElementById('carGreenDot');

      if (timerVal) timerVal.innerText = remaining;
      if (cycleInfo) cycleInfo.innerText = `CYKLUS: RED (${activeSignal.redTime}s) / GREEN (${activeSignal.greenTime}s)`;

      const isPed = userRole === 'pedestrian';

      if (state === 'RED') {
        if (timerVal) timerVal.className = "text-[130px] font-black leading-none tracking-tighter text-red-500 font-mono";
        if (statusText) {
          statusText.className = "text-[11px] font-mono text-red-500 uppercase tracking-wider mt-0.5";
          statusText.innerText = isPed ? "SEKUND DO KONCE ČERVENÉ" : "SEKUND DO ZELENÉ";
        }
        if (isPed) {
          if (pedSvg) pedSvg.style.fill = "#ef4444";
        } else {
          if (carRedDot) carRedDot.className = "w-8 h-8 rounded-full bg-red-500 shadow-[0_0_15px_rgba(239,68,68,0.9)]";
          if (carGreenDot) carGreenDot.className = "w-8 h-8 rounded-full bg-neutral-800";
        }
      } else {
        if (timerVal) timerVal.className = "text-[130px] font-black leading-none tracking-tighter text-green-500 font-mono";
        if (statusText) {
          statusText.className = "text-[11px] font-mono text-green-500 uppercase tracking-wider mt-0.5";
          statusText.innerText = isPed ? "MŮŽEŠ JÍT! SEKUND DO ČERVENÉ" : "VOLNÝ PRŮJEZD! SEKUND DO ČERVENÉ";
        }
        if (isPed) {
          if (pedSvg) pedSvg.style.fill = "#22c55e";
        } else {
          if (carRedDot) carRedDot.className = "w-8 h-8 rounded-full bg-neutral-800";
          if (carGreenDot) carGreenDot.className = "w-8 h-8 rounded-full bg-green-500 shadow-[0_0_15px_rgba(34,197,94,0.9)]";
        }
      }
    }, 500);
  </script>
</body>
</html>
