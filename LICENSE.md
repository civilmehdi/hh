<!DOCTYPE html>
<html lang="fr" dir="ltr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simulation Domotique - Allumage PC & Climatiseur par Détecteur de Mouvement</title>
  <script src="https://www.gstatic.com/antigravity/web/dev/tailwindcss.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root {
      --card: #1e293b;
      --foreground: #f8fafc;
      --muted-foreground: #94a3b8;
      --border: #334155;
      --background: #0f172a;
    }
    body {
      background-color: #0f172a !important;
      color: #f8fafc !important;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    }
    @keyframes pulse-radar {
      0% { transform: scale(0.6); opacity: 0.9; }
      100% { transform: scale(2.2); opacity: 0; }
    }
    @keyframes beam-flow {
      0% { stroke-dashoffset: 40; }
      100% { stroke-dashoffset: 0; }
    }
    @keyframes air-stream {
      0% { opacity: 0; transform: translateY(-3px) scale(0.8); }
      40% { opacity: 0.9; }
      100% { opacity: 0; transform: translateY(35px) scale(1.15); }
    }
    .radar-wave {
      animation: pulse-radar 1.5s cubic-bezier(0.2, 0.8, 0.4, 1) infinite;
    }
    .signal-beam {
      stroke-dasharray: 6 4;
      animation: beam-flow 0.8s linear infinite;
    }
    .screen-glow {
      box-shadow: 0 0 35px rgba(56, 189, 248, 0.45);
    }
    .ac-glow {
      box-shadow: 0 4px 20px rgba(56, 189, 248, 0.25);
    }
    .air-flow-1 { animation: air-stream 1.2s ease-out infinite; }
    .air-flow-2 { animation: air-stream 1.2s ease-out 0.4s infinite; }
    .air-flow-3 { animation: air-stream 1.2s ease-out 0.8s infinite; }

    .pc-rgb {
      background: linear-gradient(135deg, #06b6d4, #8b5cf6, #ec4899);
      background-size: 200% 200%;
      animation: rgb-shift 3s ease infinite;
    }
    @keyframes rgb-shift {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
  </style>
</head>
<body class="bg-transparent text-[var(--foreground)] antialiased p-3 select-none">
  <div class="bg-[var(--card)] text-[var(--foreground)] border border-[var(--border)] rounded-2xl p-5 shadow-xl max-w-4xl mx-auto">
    
    <!-- En-tête -->
    <div class="flex flex-wrap items-center justify-between border-b border-[var(--border)] pb-3 mb-4 gap-2">
      <div>
        <h2 class="text-base font-bold flex items-center gap-2">
          <span>❄️💻</span>
          <span>Système Domotique : Démarrage PC & Climatiseur par Détecteur de Mouvement</span>
        </h2>
        <p class="text-xs text-[var(--muted-foreground)]">Détection automatique d'une personne à l'entrée ➔ Allumage du PC et de la Climatisation</p>
      </div>
      <div class="flex items-center gap-2">
        <div id="statusBadge" class="px-3 py-1 rounded-full text-xs font-medium bg-amber-500/10 text-amber-500 border border-amber-500/20 flex items-center gap-1.5">
          <span class="w-2 h-2 rounded-full bg-amber-400 animate-ping"></span>
          <span>En Veille (Standby)</span>
        </div>
      </div>
    </div>

    <!-- Scène Interactive -->
    <div class="relative bg-slate-950/85 border border-[var(--border)] rounded-xl p-4 overflow-hidden min-h-[300px] flex flex-col justify-between">
      
      <!-- Mur supérieur : Climatiseur & Capteur PIR -->
      <div class="grid grid-cols-12 items-start relative z-10 w-full mb-4">
        
        <!-- Entrée / Porte -->
        <div class="col-span-3 text-[11px] text-slate-400 flex items-center gap-1">
          <span>🚪 Entrée de la pièce</span>
        </div>

        <!-- Capteur PIR au centre -->
        <div class="col-span-5 flex flex-col items-center justify-center relative">
          <div class="relative flex flex-col items-center">
            <div class="w-10 h-10 rounded-xl bg-slate-800 border-2 border-slate-600 flex items-center justify-center relative shadow-lg">
              <div id="sensorDome" class="w-6 h-6 rounded-full bg-slate-300 border border-slate-400 flex items-center justify-center transition-colors duration-300">
                <div id="sensorLed" class="w-2 h-2 rounded-full bg-emerald-500 transition-colors"></div>
              </div>
              <!-- Ondes Radar -->
              <div id="radarWave" class="hidden absolute -inset-4 rounded-full border-2 border-cyan-400 radar-wave pointer-events-none"></div>
            </div>
            <span class="text-[10px] text-cyan-400 font-semibold mt-1">Capteur PIR</span>
          </div>

          <!-- Signal de transmission -->
          <div id="signalBranch" class="hidden w-full flex justify-between items-center text-[9px] text-sky-300 font-mono mt-2 bg-sky-950/70 border border-sky-600/40 rounded px-2 py-0.5 animate-pulse">
            <span>⚡ Signal IR (Clim)</span>
            <span>⚡ Signal Wake-on-LAN (PC)</span>
          </div>
        </div>

        <!-- Climatiseur (Haut Droite) -->
        <div class="col-span-4 flex flex-col items-end">
          <div id="acUnit" class="w-48 bg-slate-200 border-2 border-slate-300 rounded-lg p-2 text-slate-800 relative shadow transition-all duration-500">
            <div class="flex justify-between items-center text-[10px] mb-1">
              <span class="font-bold tracking-wider text-slate-700">CLIMATISEUR</span>
              <div id="acDisplay" class="bg-black/90 text-slate-500 px-2 py-0.5 rounded font-mono text-[11px] font-bold border border-slate-700 transition-colors">
                --°C
              </div>
            </div>

            <div class="flex justify-between items-center">
              <div class="flex items-center gap-1">
                <div id="acPowerLed" class="w-1.5 h-1.5 rounded-full bg-slate-400"></div>
                <span id="acStateText" class="text-[9px] text-slate-500">Arrêt (OFF)</span>
              </div>
            </div>

            <!-- Volet d'air -->
            <div class="mt-2 pt-1 border-t border-slate-300 relative">
              <div id="acFlap" class="w-full h-1.5 bg-slate-400 rounded-full transition-all duration-700 origin-top"></div>
            </div>

            <!-- Flux d'air frais -->
            <div id="acAirFlow" class="hidden absolute -bottom-16 left-0 right-0 flex justify-around pointer-events-none">
              <div class="flex flex-col items-center air-flow-1">
                <span class="text-sky-400 text-xs">❄️</span>
                <div class="w-1 h-8 bg-gradient-to-b from-sky-400 to-transparent rounded-full"></div>
              </div>
              <div class="flex flex-col items-center air-flow-2">
                <span class="text-cyan-300 text-xs">💨</span>
                <div class="w-1.5 h-10 bg-gradient-to-b from-cyan-400 to-transparent rounded-full"></div>
              </div>
              <div class="flex flex-col items-center air-flow-3">
                <span class="text-sky-400 text-xs">❄️</span>
                <div class="w-1 h-8 bg-gradient-to-b from-sky-400 to-transparent rounded-full"></div>
              </div>
            </div>
          </div>
          <span class="text-[10px] text-sky-400 font-medium mt-1">Climatiseur Split</span>
        </div>

      </div>

      <!-- Bas de scène : Personne & Bureau PC -->
      <div class="flex items-end justify-between relative z-10 w-full pt-4">
        
        <!-- Personnage et porte -->
        <div class="flex flex-col items-center">
          <div class="w-24 h-36 border-2 border-slate-700 rounded-t-lg relative bg-slate-900/60 flex items-end justify-center pb-2">
            <div class="absolute top-1 left-2 w-2 h-2 rounded-full bg-slate-600"></div>
            
            <div id="personFigure" class="transition-all duration-700 transform -translate-x-12 opacity-30 flex flex-col items-center">
              <div class="w-7 h-7 rounded-full bg-amber-200 border border-amber-300 shadow-sm relative">
                <div class="w-1.5 h-1.5 bg-slate-800 rounded-full absolute top-2 left-1.5"></div>
              </div>
              <div class="w-10 h-12 bg-indigo-600 rounded-t-lg mt-0.5 flex justify-center">
                <div class="w-2 h-4 bg-indigo-400 rounded-full mt-1.5"></div>
              </div>
              <div class="flex gap-1.5">
                <div class="w-3.5 h-10 bg-slate-800 rounded-b"></div>
                <div class="w-3.5 h-10 bg-slate-800 rounded-b"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Message d'état -->
        <div class="flex-1 px-4 text-center">
          <div id="eventMessage" class="text-xs text-slate-400 font-mono bg-slate-900/80 border border-slate-800 rounded-lg p-2.5 shadow">
            En attente d'un mouvement à l'entrée...
          </div>
        </div>

        <!-- Bureau et PC -->
        <div class="flex flex-col items-center">
          <div class="flex items-end gap-2">
            <!-- Moniteur -->
            <div class="flex flex-col items-center">
              <div id="monitorBezel" class="w-32 h-24 bg-slate-900 border-4 border-slate-700 rounded-lg p-1.5 flex flex-col justify-between transition-all duration-500">
                <div id="monitorScreen" class="w-full h-full bg-black rounded flex flex-col items-center justify-center text-center p-1 transition-all duration-700">
                  <span id="screenSleepText" class="text-[9px] text-slate-600 font-mono">💤 VEILLE</span>
                  
                  <div id="desktopView" class="hidden w-full h-full flex flex-col justify-between p-1">
                    <div class="flex justify-between items-center text-[7px] text-sky-300 bg-sky-950/60 rounded px-1">
                      <span>Windows 11</span>
                      <span id="clockDisplay">13:00</span>
                    </div>
                    <div class="flex items-center justify-center gap-1 my-auto">
                      <span class="text-xs">👋</span>
                      <span class="text-[8px] font-bold text-sky-200">Bienvenue ! PC Prêt</span>
                    </div>
                    <div class="w-full h-1 bg-sky-500/40 rounded-full overflow-hidden">
                      <div class="w-full h-full bg-sky-400 animate-pulse"></div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="w-3 h-3 bg-slate-700"></div>
              <div class="w-12 h-1 bg-slate-600 rounded"></div>
            </div>

            <!-- Boîtier PC -->
            <div class="flex flex-col items-center">
              <div id="pcCase" class="w-12 h-26 bg-slate-900 border-2 border-slate-700 rounded-md p-1.5 flex flex-col justify-between transition-all duration-300">
                <div class="flex justify-end">
                  <div id="powerBtnLed" class="w-2 h-2 rounded-full bg-slate-700 transition-colors"></div>
                </div>
                <div class="space-y-1 my-auto">
                  <div id="fan1" class="w-7 h-7 rounded-full border border-slate-700 mx-auto transition-all"></div>
                  <div id="fan2" class="w-7 h-7 rounded-full border border-slate-700 mx-auto transition-all"></div>
                </div>
                <div id="pcRgbStrip" class="w-full h-1 bg-slate-800 rounded"></div>
              </div>
              <span class="text-[10px] text-slate-400 mt-1 font-mono">PC Tour</span>
            </div>
          </div>
          <!-- Surface du bureau -->
          <div class="w-48 h-2 bg-amber-900/70 border-t border-amber-600 rounded-sm mt-1"></div>
        </div>

      </div>

    </div>

    <!-- Boutons d'action -->
    <div class="mt-4 flex flex-wrap items-center justify-between gap-3">
      <div class="flex items-center gap-2">
        <button id="btnEnter" onclick="triggerSystem()" class="px-4 py-2 rounded-lg bg-indigo-600 hover:bg-indigo-500 text-white font-medium text-xs flex items-center gap-1.5 shadow transition-all active:scale-95">
          <span>🚶‍♂️ Simuler l'entrée (Allumer PC & Clim)</span>
        </button>
        <button id="btnReset" onclick="resetSystem()" class="px-3 py-2 rounded-lg border border-[var(--border)] hover:bg-slate-800 text-xs font-medium transition-colors">
          <span>🔄 Réinitialiser</span>
        </button>
      </div>

      <button onclick="downloadHTMLFile()" class="px-4 py-2 rounded-lg bg-emerald-600 hover:bg-emerald-500 text-white font-semibold text-xs flex items-center gap-1.5 shadow transition-all active:scale-95">
        <span>💾 Télécharger le fichier (HTML)</span>
      </button>
    </div>

    <!-- Explications Techniques -->
    <div class="mt-4 pt-4 border-t border-[var(--border)] grid grid-cols-1 md:grid-cols-3 gap-3 text-xs">
      
      <div class="bg-[var(--background)]/50 p-3 rounded-lg border border-[var(--border)]">
        <div class="font-bold text-cyan-400 mb-1 flex items-center gap-1">
          <span>1️⃣</span> <span>Capteur de Mouvement (PIR)</span>
        </div>
        <p class="text-[var(--muted-foreground)] text-[11px] leading-relaxed">
          Le capteur (ex. <strong>HC-SR501</strong> ou Zigbee <strong>Sonoff / Tuya</strong>) détecte la présence humaine dès qu'une personne franchit la porte.
        </p>
      </div>

      <div class="bg-[var(--background)]/50 p-3 rounded-lg border border-[var(--border)]">
        <div class="font-bold text-sky-400 mb-1 flex items-center gap-1">
          <span>2️⃣</span> <span>Allumage Climatiseur (Infrarouge)</span>
        </div>
        <p class="text-[var(--muted-foreground)] text-[11px] leading-relaxed">
          Un <strong>Smart IR Blaster</strong> (Tuya / Broadlink ou ESP32) reproduit le signal de la télécommande et met la clim en route à <strong>22°C</strong>.
        </p>
      </div>

      <div class="bg-[var(--background)]/50 p-3 rounded-lg border border-[var(--border)]">
        <div class="font-bold text-indigo-400 mb-1 flex items-center gap-1">
          <span>3️⃣</span> <span>Démarrage du PC (Wake-on-LAN)</span>
        </div>
        <p class="text-[var(--muted-foreground)] text-[11px] leading-relaxed">
          Envoi d'un paquet <strong>Magic Packet (WOL)</strong> par le réseau local (Wi-Fi/Ethernet) : la carte mère allume le PC et les écrans sans fil.
        </p>
      </div>

    </div>

  </div>

  <script>
    let isRunning = false;

    function triggerSystem() {
      if (isRunning) return;
      isRunning = true;

      const person = document.getElementById('personFigure');
      const sensorDome = document.getElementById('sensorDome');
      const sensorLed = document.getElementById('sensorLed');
      const radarWave = document.getElementById('radarWave');
      const signalBranch = document.getElementById('signalBranch');
      const statusBadge = document.getElementById('statusBadge');
      const eventMessage = document.getElementById('eventMessage');

      const acUnit = document.getElementById('acUnit');
      const acDisplay = document.getElementById('acDisplay');
      const acPowerLed = document.getElementById('acPowerLed');
      const acStateText = document.getElementById('acStateText');
      const acFlap = document.getElementById('acFlap');
      const acAirFlow = document.getElementById('acAirFlow');

      const monitorScreen = document.getElementById('monitorScreen');
      const screenSleepText = document.getElementById('screenSleepText');
      const desktopView = document.getElementById('desktopView');
      const powerBtnLed = document.getElementById('powerBtnLed');
      const pcRgbStrip = document.getElementById('pcRgbStrip');
      const fan1 = document.getElementById('fan1');
      const fan2 = document.getElementById('fan2');

      // 1. Personne entre
      person.classList.remove('-translate-x-12', 'opacity-30');
      person.classList.add('translate-x-0', 'opacity-100');
      eventMessage.innerText = "🚶‍♂️ Une personne entre dans la pièce...";

      setTimeout(() => {
        // 2. Détection par le capteur
        sensorDome.classList.remove('bg-slate-300');
        sensorDome.classList.add('bg-rose-400');
        sensorLed.classList.remove('bg-emerald-500');
        sensorLed.classList.add('bg-rose-600');
        radarWave.classList.remove('hidden');

        statusBadge.className = "px-3 py-1 rounded-full text-xs font-medium bg-rose-500/10 text-rose-400 border border-rose-500/20 flex items-center gap-1.5";
        statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-rose-500 animate-ping"></span><span>⚠️ Mouvement Détecté ! Envoi des commandes...</span>';
        
        signalBranch.classList.remove('hidden');
        eventMessage.innerText = "📡 Envoi du paquet Wake-on-LAN au PC + signal Infrarouge à la clim...";
      }, 500);

      setTimeout(() => {
        // 3. Climatiseur Allumé
        acUnit.classList.add('ac-glow', 'border-sky-400');
        acDisplay.classList.remove('text-slate-500');
        acDisplay.classList.add('text-cyan-400', 'shadow-[0_0_10px_#06b6d4]');
        acDisplay.innerText = "22°C ❄️";

        acPowerLed.className = "w-1.5 h-1.5 rounded-full bg-emerald-400 shadow-[0_0_6px_#34d399]";
        acStateText.innerText = "Actif (Froid ON)";
        acStateText.className = "text-[9px] text-emerald-400 font-bold";

        acFlap.style.transform = "rotate(30deg) translateY(4px)";
        acAirFlow.classList.remove('hidden');

        // 4. PC Allumé
        powerBtnLed.className = "w-2 h-2 rounded-full bg-cyan-400 shadow-[0_0_8px_#38bdf8]";
        fan1.classList.add('pc-rgb', 'border-cyan-400');
        fan2.classList.add('pc-rgb', 'border-pink-400');
        pcRgbStrip.classList.add('pc-rgb');

        monitorScreen.classList.remove('bg-black');
        monitorScreen.classList.add('bg-gradient-to-br', 'from-slate-900', 'via-sky-950', 'to-indigo-950', 'screen-glow');
        screenSleepText.classList.add('hidden');
        desktopView.classList.remove('hidden');

        statusBadge.className = "px-3 py-1 rounded-full text-xs font-medium bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 flex items-center gap-1.5";
        statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-emerald-400"></span><span>✅ PC & Climatiseur allumés avec succès !</span>';

        eventMessage.innerText = "🎉 Bienvenue ! Le PC est prêt et la clim rafraîchit la pièce à 22°C.";
      }, 1300);
    }

    function resetSystem() {
      isRunning = false;

      const person = document.getElementById('personFigure');
      const sensorDome = document.getElementById('sensorDome');
      const sensorLed = document.getElementById('sensorLed');
      const radarWave = document.getElementById('radarWave');
      const signalBranch = document.getElementById('signalBranch');
      const statusBadge = document.getElementById('statusBadge');
      const eventMessage = document.getElementById('eventMessage');

      const acUnit = document.getElementById('acUnit');
      const acDisplay = document.getElementById('acDisplay');
      const acPowerLed = document.getElementById('acPowerLed');
      const acStateText = document.getElementById('acStateText');
      const acFlap = document.getElementById('acFlap');
      const acAirFlow = document.getElementById('acAirFlow');

      const monitorScreen = document.getElementById('monitorScreen');
      const screenSleepText = document.getElementById('screenSleepText');
      const desktopView = document.getElementById('desktopView');
      const powerBtnLed = document.getElementById('powerBtnLed');
      const pcRgbStrip = document.getElementById('pcRgbStrip');
      const fan1 = document.getElementById('fan1');
      const fan2 = document.getElementById('fan2');

      person.classList.add('-translate-x-12', 'opacity-30');
      person.classList.remove('translate-x-0', 'opacity-100');

      sensorDome.className = "w-6 h-6 rounded-full bg-slate-300 border border-slate-400 flex items-center justify-center transition-colors duration-300";
      sensorLed.className = "w-2 h-2 rounded-full bg-emerald-500 transition-colors";
      radarWave.classList.add('hidden');
      signalBranch.classList.add('hidden');

      acUnit.className = "w-48 bg-slate-200 border-2 border-slate-300 rounded-lg p-2 text-slate-800 relative shadow transition-all duration-500";
      acDisplay.className = "bg-black/90 text-slate-500 px-2 py-0.5 rounded font-mono text-[11px] font-bold border border-slate-700 transition-colors";
      acDisplay.innerText = "--°C";
      acPowerLed.className = "w-1.5 h-1.5 rounded-full bg-slate-400";
      acStateText.innerText = "Arrêt (OFF)";
      acStateText.className = "text-[9px] text-slate-500";
      acFlap.style.transform = "none";
      acAirFlow.classList.add('hidden');

      monitorScreen.className = "w-full h-full bg-black rounded flex flex-col items-center justify-center text-center p-1 transition-all duration-700";
      screenSleepText.classList.remove('hidden');
      desktopView.classList.add('hidden');
      powerBtnLed.className = "w-2 h-2 rounded-full bg-slate-700 transition-colors";
      fan1.className = "w-7 h-7 rounded-full border border-slate-700 mx-auto transition-all";
      fan2.className = "w-7 h-7 rounded-full border border-slate-700 mx-auto transition-all";
      pcRgbStrip.className = "w-full h-1 bg-slate-800 rounded";

      statusBadge.className = "px-3 py-1 rounded-full text-xs font-medium bg-amber-500/10 text-amber-500 border border-amber-500/20 flex items-center gap-1.5";
      statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-amber-400 animate-ping"></span><span>En Veille (Standby)</span>';

      eventMessage.innerText = "En attente d'un mouvement à l'entrée...";
    }

    function downloadHTMLFile() {
      const doctype = "<!DOCTYPE html>\n";
      const fullHtml = doctype + document.documentElement.outerHTML;
      const blob = new Blob([fullHtml], { type: 'text/html;charset=utf-8' });
      const downloadLink = document.createElement('a');
      downloadLink.href = URL.createObjectURL(blob);
      downloadLink.download = 'simulation_pc_climatiseur_pir.html';
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);
    }
  </script>
</body>
</html>
