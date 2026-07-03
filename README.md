<!DOCTYPE html> 
<html lang="pl">
<head>
<meta charset="UTF-8">
<title>Kasjer Simulator 3D - Korpo Imperium PRO</title>

<style>
body {
    margin: 0;
    overflow: hidden;
    font-family: 'Courier New', Courier, monospace;
    background-color: #05050a;
    user-select: none;
}
/* Retro pixel-art look: scale internal canvas up with nearest-neighbor */
body > canvas {
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edges;
    image-rendering: crisp-edges;
    filter: contrast(1.08) saturate(1.15);
}

#ui-container {
    position: absolute;
    left: 20px;
    top: 20px;
    background: #222533;
    color: #fff;
    padding: 20px;
    font-size: 16px;
    font-weight: bold;
    border: 4px solid #12141c;
    box-shadow: 6px 6px 0px #000;
    line-height: 1.6;
    min-width: 280px;
    z-index: 100;
}

#shop-container {
    position: absolute;
    right: 20px;
    top: 20px;
    background: #222533;
    color: #fff;
    padding: 8px;
    border: 3px solid #12141c;
    box-shadow: 4px 4px 0px #000;
    width: 410px;
    z-index: 100;
}

.tabs {
    display: flex;
    gap: 4px;
    margin-bottom: 8px;
}

.tab-btn {
    flex: 1;
    background: #ffec27;
    color: #000;
    border: 2px solid #12141c;
    padding: 5px 2px;
    font-family: 'Courier New', Courier, monospace;
    font-weight: bold;
    font-size: 10px;
    cursor: pointer;
    text-align: center;
}

.tab-btn.active {
    background: #00e436;
    color: #000;
}

.shop-content {
    max-height: 450px; 
    overflow-y: auto;
}

.shop-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #1a1c2c;
    padding: 5px 6px;
    margin: 4px 0;
    border: 1px solid #3a4466;
    font-size: 11px;
}

.shop-btn {
    background: #00e436;
    color: #000;
    border: 1px solid #000;
    padding: 3px 6px;
    font-weight: bold;
    font-size: 10px;
    font-family: 'Courier New', Courier, monospace;
    cursor: pointer;
    min-width: 105px;
    text-align: center;
}
.shop-btn:disabled {
    background: #5f574f;
    color: #ab9b8e;
    cursor: not-allowed;
}

.ui-line {
    display: flex;
    justify-content: space-between;
    margin-bottom: 5px;
    border-bottom: 2px dashed #3a4466;
    padding-bottom: 5px;
}

#dymek {
    position: absolute;
    top: 30px;
    left: 50%;
    transform: translateX(-50%);
    background: #fff;
    color: #000;
    padding: 12px 35px;
    font-size: 20px;
    font-weight: bold;
    box-shadow: 6px 6px 0px #000;
    border: 4px solid #47af4c;
    text-align: center;
    z-index: 100;
    max-width: 600px;
}

.progress-bar-container {
    width: 100%;
    background: #12141c;
    height: 14px;
    border: 3px solid #fff;
    margin-top: 5px;
    box-sizing: border-box;
}

#progress-bar-fill {
    width: 100%;
    height: 100%;
    background: #00e436;
    transition: width 0.1s linear;
}

#energy-bar-fill {
    width: 100%;
    height: 100%;
    background: #29adff;
    transition: width 0.3s ease;
}

.modal-screen {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: #12141c;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 999;
    display: none;
    text-align: center;
}

#game-over-screen { color: #ff004d; font-size: 36px; font-weight: bold; }
#day-clear-screen { color: #00e436; font-size: 36px; font-weight: bold; }

.action-btn {
    margin-top: 25px;
    padding: 15px 40px;
    font-size: 20px;
    font-family: 'Courier New', Courier, monospace;
    background: #00e436;
    color: #000;
    border: 4px solid #000;
    box-shadow: 4px 4px 0px #000;
    cursor: pointer;
    font-weight: bold;
}

#controls-hint {
display: none;
    position: absolute;
    bottom: 15px;
    left: 50%;
    transform: translateX(-50%);
    background: #222533;
    color: #ab9b8e;
    padding: 10px 20px;
    font-size: 12px;
    border: 3px solid #12141c;
    box-shadow: 3px 3px 0px #000;
    pointer-events: none;
    z-index: 100;
    text-align: center;
    line-height: 1.4;
}

#kredyt-box {
    margin-top: 15px;
    background: #1a1c2c;
    border: 2px solid #ff004d;
    padding: 10px;
    font-size: 12px;
    text-align: center;
}

.kredyt-btn {
    background: #ff004d;
    color: white;
    border: none;
    padding: 5px 10px;
    font-family: monospace;
    font-weight: bold;
    cursor: pointer;
    margin-top: 5px;
}

/* Style dedykowane dla systemu tasków */
#task-alert-box {
    background: #ff004d;
    color: #fff;
    padding: 10px;
    text-align: center;
    font-weight: bold;
    animation: blinker 1s linear infinite;
    margin-bottom: 10px;
    display: none;
}
@keyframes blinker {
    50% { opacity: 0.5; }
}

#start-menu {
    position: fixed;
    top: 0; left: 0;
    width: 100vw; height: 100vh;
    background:
      radial-gradient(ellipse at 50% 110%, #2a1a4a 0%, transparent 55%),
      radial-gradient(circle at 18% 25%, rgba(0,228,212,0.18), transparent 55%),
      radial-gradient(circle at 82% 75%, rgba(255,0,85,0.22), transparent 55%),
      linear-gradient(180deg, #05050a 0%, #100823 55%, #1a0b2e 100%);
    display: flex; flex-direction: column;
    justify-content: center; align-items: center;
    z-index: 9999;
    overflow: hidden;
    image-rendering: pixelated;
}
/* Pikselowa siatka tła */
#start-menu::before {
    content: ""; position: absolute; inset: 0;
    background-image:
      linear-gradient(rgba(0,255,204,0.08) 2px, transparent 2px),
      linear-gradient(90deg, rgba(0,255,204,0.08) 2px, transparent 2px);
    background-size: 32px 32px;
    pointer-events: none;
    animation: gridMove 6s linear infinite;
    mask-image: radial-gradient(ellipse at center, black 30%, transparent 80%);
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 80%);
}
/* Skanlinie CRT */
#start-menu::after {
    content: ""; position: absolute; inset: 0;
    background: repeating-linear-gradient(
      to bottom,
      rgba(0,0,0,0.18) 0px,
      rgba(0,0,0,0.18) 2px,
      transparent 2px,
      transparent 4px
    );
    pointer-events: none;
    z-index: 2;
    mix-blend-mode: multiply;
}
@keyframes gridMove {
    from { background-position: 0 0, 0 0; }
    to   { background-position: 32px 32px, 32px 32px; }
}
/* Latające pixelowe gwiazdki */
.pix-star {
    position: absolute;
    width: 4px; height: 4px;
    background: #ffec27;
    box-shadow: 0 0 0 1px #000;
    animation: floatStar linear infinite;
    z-index: 1;
}
@keyframes floatStar {
    0%   { transform: translateY(110vh) scale(1); opacity: 0; }
    10%  { opacity: 1; }
    90%  { opacity: 1; }
    100% { transform: translateY(-10vh) scale(1); opacity: 0; }
}
.menu-box {
    position: relative;
    background:
      linear-gradient(180deg, #2a2d4a 0%, #1a1c2c 50%, #12141c 100%);
    padding: 44px 56px 38px 56px;
    border: 4px solid #000;
    box-shadow:
      0 0 0 4px #ffec27,
      0 0 0 8px #000,
      12px 12px 0 #000,
      0 0 80px rgba(0,228,212,0.35);
    text-align: center; min-width: 460px;
    z-index: 3;
    image-rendering: pixelated;
}
/* Pikselowy ornament w rogach */
.menu-box::before, .menu-box::after {
    content: ""; position: absolute;
    width: 18px; height: 18px;
    background:
      linear-gradient(#ff0055,#ff0055) 0 0/6px 6px no-repeat,
      linear-gradient(#00e4d4,#00e4d4) 6px 0/6px 6px no-repeat,
      linear-gradient(#ffec27,#ffec27) 0 6px/6px 6px no-repeat,
      linear-gradient(#000,#000) 6px 6px/6px 6px no-repeat,
      #1a1c2c;
}
.menu-box::before { top: -4px; left: -4px; }
.menu-box::after  { bottom: -4px; right: -4px; transform: rotate(180deg); }

.menu-title {
    font-size: 52px; color: #ffec27; margin: 0 0 6px 0;
    text-transform: uppercase; font-weight: bold; letter-spacing: 4px;
    text-shadow:
      3px 0 0 #ff0055,
      -3px 0 0 #00e4d4,
      3px 3px 0 #000,
      6px 6px 0 #000;
    animation: titleGlitch 3.6s ease-in-out infinite;
    line-height: 1;
}
@keyframes titleGlitch {
    0%, 92%, 100% { transform: translate(0,0); }
    93% { transform: translate(-2px, 1px); }
    95% { transform: translate(2px, -1px); }
    97% { transform: translate(-1px, 2px); }
}
.menu-subtitle {
    font-size: 14px; color: #ff77a8; margin-bottom: 28px;
    font-weight: bold; text-transform: uppercase; letter-spacing: 6px;
    text-shadow: 2px 2px 0 #000;
    padding: 4px 8px;
    border-top: 2px dashed #ff77a8;
    border-bottom: 2px dashed #ff77a8;
    display: inline-block;
}
.menu-label {
    color: #00e4d4; font-size: 13px; letter-spacing: 3px;
    margin-bottom: 16px; text-transform: uppercase;
    text-shadow: 2px 2px 0 #000;
    animation: labelBlink 1.2s steps(2) infinite;
}
@keyframes labelBlink {
    0%, 49% { opacity: 1; }
    50%, 100% { opacity: 0.55; }
}
.menu-modes {
    display: flex; flex-direction: column; gap: 14px; align-items: stretch;
}
.btn-tryb {
    position: relative;
    background:
      linear-gradient(180deg, #1c1e2e 0%, #12141c 100%);
    color: #fff;
    border: 3px solid #00e4d4;
    padding: 16px 22px;
    font-size: 17px;
    font-family: 'Courier New', Courier, monospace;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 5px 5px 0 #000;
    text-align: left;
    transition: all 0.1s steps(2);
    text-transform: uppercase;
    letter-spacing: 1px;
    image-rendering: pixelated;
}
.btn-tryb .tryb-desc {
    display: block; font-size: 11px; color: #ab9b8e;
    margin-top: 6px; font-weight: normal; letter-spacing: 0;
    text-transform: none;
}
.btn-tryb.latwy        { border-color: #00e436; color: #00e436; }
.btn-tryb.latwy:hover  { background: linear-gradient(180deg,#00e436 0%, #008751 100%); color: #000; }
.btn-tryb.normalny     { border-color: #ff0055; color: #ff0055; }
.btn-tryb.normalny:hover { background: linear-gradient(180deg,#ff0055 0%, #7e2553 100%); color: #fff; }
.btn-tryb:hover  { transform: translate(-2px, -2px); box-shadow: 7px 7px 0 #000; }
.btn-tryb:hover .tryb-desc { color: rgba(0,0,0,0.78); }
.btn-tryb.normalny:hover .tryb-desc { color: rgba(255,255,255,0.85); }
.btn-tryb:active { transform: translate(3px, 3px); box-shadow: 2px 2px 0 #000; }
.menu-foot {
    margin-top: 24px; color: #5f574f; font-size: 10px;
    letter-spacing: 3px; text-transform: uppercase;
    border-top: 2px solid #3a4466;
    padding-top: 10px;
}
.menu-foot .blink { color: #ffec27; animation: labelBlink 0.8s steps(2) infinite; }


#black-market {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: #0a0014;
    color: #00e436;
    border: 4px solid #ff004d;
    box-shadow: 8px 8px 0px #000, 0 0 40px #ff004d;
    padding: 20px;
    width: 520px;
    z-index: 200;
    font-family: 'Courier New', Courier, monospace;
    display: none;
    animation: bmFlicker 2s infinite;
}
@keyframes bmFlicker { 0%,97%,100% { opacity:1; } 98% { opacity:0.7; } }
#black-market h2 {
    color: #ff004d; margin: 0 0 4px 0; font-size: 22px; text-align:center;
    text-shadow: 2px 2px 0 #000;
}
#black-market .bm-sub { color:#ffec27; text-align:center; font-size:11px; margin-bottom:12px; }
.burger-item {
    display:flex; justify-content:space-between; align-items:center;
    background:#1a0014; border:2px solid #ff004d; padding:8px 10px; margin:6px 0;
    font-size:12px;
}
.burger-item .b-info { color:#fff; }
.burger-item .b-info b { color:#ffec27; }
.burger-item .b-desc { color:#83769c; font-size:10px; margin-top:2px; }
.burger-btn {
    background:#ff004d; color:#fff; border:2px solid #000; padding:6px 10px;
    font-family:'Courier New',monospace; font-weight:bold; font-size:11px;
    cursor:pointer; min-width:90px;
}
.burger-btn:hover { background:#ffec27; color:#000; }
#bm-close-hint { text-align:center; color:#ab9b8e; font-size:11px; margin-top:10px; }
.burger-item.locked { opacity:0.55; filter:grayscale(0.6); }
.burger-item.locked .burger-btn { background:#444; color:#999; cursor:not-allowed; }
.burger-item .b-lock { color:#ff004d; font-size:10px; margin-top:2px; font-weight:bold; }

#magazyn-warn {
    position:absolute; left:50%; top:30%; transform:translate(-50%,-50%);
    background:#1a0000; color:#ff004d; border:4px solid #ff004d;
    box-shadow:8px 8px 0 #000, 0 0 30px #ff004d;
    padding:20px 30px; font-family:'Courier New',monospace; font-weight:bold;
    font-size:18px; text-align:center; z-index:200; display:none;
    animation: bmFlicker 0.8s infinite;
}
#magazyn-warn .mw-sub { color:#ffec27; font-size:12px; margin-top:8px; }
#hint-obrot {
    position:absolute; bottom:8px; left:50%; transform:translateX(-50%);
    color:#ffec27; font-family:'Courier New',monospace; font-size:12px;
    background:#000a; padding:6px 12px; border:2px solid #ffec27; z-index:50;
}
#btn-jak-grac {
    position:fixed; top:8px; right:8px; z-index:300;
    background:#ffec27; color:#000; border:2px solid #000;
    box-shadow:2px 2px 0 #000;
    font-family:'Courier New',monospace; font-weight:bold; font-size:10px;
    padding:3px 6px; cursor:pointer;
}
#btn-jak-grac:hover { background:#ff004d; color:#fff; }

#radio-btn {
    position:fixed; bottom:12px; right:12px; z-index:300;
    background:#3a2210; color:#ffec27; border:3px solid #ffec27;
    box-shadow:3px 3px 0 #000;
    font-family:'Courier New',monospace; font-weight:bold; font-size:12px;
    padding:6px 10px; cursor:pointer; display:flex; align-items:center; gap:6px;
}
#radio-btn:hover { background:#5c3a1e; }
#radio-btn .radio-led {
    width:8px; height:8px; border-radius:50%; background:#444; border:1px solid #222; display:inline-block;
}
#radio-btn.playing .radio-led { background:#00e436; box-shadow:0 0 6px #00e436; }
#radio-btn.playing { color:#00e436; border-color:#00e436; }
#panel-pomoc {
    display:none; position:fixed; top:60px; right:12px; z-index:300;
    width:380px; max-height:80vh; overflow-y:auto;
    background:#1a0014; color:#fff; border:3px solid #ffec27;
    box-shadow:6px 6px 0 #000;
    font-family:'Courier New',monospace; font-size:12px;
    padding:14px 16px;
}
#panel-pomoc .ph-title {
    font-size:18px; color:#ffec27; text-align:center;
    border-bottom:2px solid #ff004d; padding-bottom:6px; margin-bottom:10px;
}
#panel-pomoc .ph-sek {
    background:#000; border:2px solid #ff004d; padding:8px 10px;
    margin:8px 0; line-height:1.5;
}
#panel-pomoc .ph-sek b { color:#ffec27; }
#panel-pomoc .ph-close {
    text-align:center; color:#83769c; margin-top:8px; cursor:pointer;
    padding:6px; border:2px dashed #83769c;
}
#panel-pomoc .ph-close:hover { color:#fff; border-color:#fff; }

</style>
</head>
<body>

<div id="start-menu">
    <div id="pix-stars"></div>
    <div class="menu-box">
        <div class="menu-title">KASJER</div>
        <div class="menu-subtitle">★ Korpo Imperium PRO ★</div>
        <div class="menu-label">► Wybierz tryb gry ◄</div>
        <div class="menu-modes">
            <button class="btn-tryb latwy" onclick="uruchomGre('latwy')">
                ▶ TRYB ŁATWY
                <span class="tryb-desc">Większa cierpliwość klientów · 7 opinii na start · bez złodziei · bez sprzątania</span>
            </button>
            <button class="btn-tryb normalny" onclick="uruchomGre('normalny')">
                ▶ TRYB NORMALNY
                <span class="tryb-desc">Pełna wersja korpo — wszystkie zdarzenia, 5 opinii na start</span>
            </button>
        </div>
        <div class="menu-foot">v1.1 · <span class="blink">PRESS START</span> · Korpo Imperium</div>
    </div>
</div>
<script>
(function(){
    const wrap = document.getElementById("pix-stars");
    if(!wrap) return;
    const kolory = ["#ffec27","#00e4d4","#ff77a8","#fff","#ff0055"];
    for(let i=0;i<28;i++){
        const s = document.createElement("div");
        s.className = "pix-star";
        s.style.left = Math.random()*100 + "vw";
        s.style.background = kolory[i % kolory.length];
        const dur = 4 + Math.random()*7;
        s.style.animationDuration = dur + "s";
        s.style.animationDelay = (-Math.random()*dur) + "s";
        const size = 3 + Math.floor(Math.random()*3);
        s.style.width = size+"px"; s.style.height = size+"px";
        wrap.appendChild(s);
    }
})();
</script>

<div id="ui-container">
    <div class="ui-line">📅 <span>STANOWISKO:</span> <span id="val-dzien-etykieta" style="color:#ffec27">DZIEŃ 1</span></div>
    <div class="ui-line">🏆 <span>Prestiż sklepu:</span> <span id="val-prestiz" style="color:#ffa300">Poziom 1</span></div>
    <div class="ui-line">🚶 <span>Klienci do końca:</span> <span id="val-do-konca" style="color:white">10</span></div>
    <div class="ui-line">🛒 <span>Obsłużeni:</span> <span id="val-wynik" style="color:white">0</span></div>
    <div class="ui-line">💰 <span>Kasa ($):</span> <span id="val-zarobek" style="color:#00e436">0.00 zł</span></div>
    <div class="ui-line">🔥 <span>Combo Mnożnik:</span> <span id="val-combo" style="color:#ff77a8">x1</span></div>
    <div class="ui-line">⭐ <span>Opinia:</span> <span id="val-gwiazdki" style="color:#ffec27">⭐⭐⭐⭐⭐</span></div>
    <div class="ui-line">🧾 <span>Do rachunków:</span> <span id="val-rachunki-licznik" style="color:#ff77a8">3 dni</span></div>
    
    <div style="margin-top: 10px;">⏱️ Cierpliwość klienta:</div>
    <div class="progress-bar-container">
        <div id="progress-bar-fill"></div>
    </div>

    <div style="margin-top: 8px;">⚡ Zmęczenie kasjera:</div>
    <div class="progress-bar-container">
        <div id="energy-bar-fill"></div>
    </div>

    <div id="kredyt-box">
        🚨 Szybka pożyczka korpo:<br>
        Dostajesz <b>1000 zł</b> / Rata: <b>150 zł/dzień</b>
        <button class="kredyt-btn" onclick="wzmijKredyt()">WEŹ KREDYT</button>
    </div>
</div>

<div id="shop-container">
    <div class="tabs">
        <div id="tab-upgrades" class="tab-btn active" onclick="switchTab('upgrades')">🚀 ULEPSZENIA</div>
        <div id="tab-custom" class="tab-btn" onclick="switchTab('custom')">🎨 WYGLĄD</div>
        <div id="tab-tasks" class="tab-btn" style="background: #ff77a8" onclick="switchTab('tasks')">📋 TASKI (<span id="task-tab-counter">0</span>)</div>
        <div id="tab-stats" class="tab-btn" style="background: #29adff; color: #fff" onclick="switchTab('stats')">📊 STATY</div>
        <div id="tab-career" class="tab-btn" style="background: #a020f0; color: #fff" onclick="switchTab('career')">👔 KARIERA</div>
    </div>
    
    <div id="content-upgrades" class="shop-content">
        <div class="shop-item"><div id="txt-up-tasma">🚀 1. Taśma (Lvl 1)</div><button class="shop-btn" id="btn-up-tasma" onclick="upgradeTasma()">Lvl 2: 200zł</button></div>
        <div class="shop-item"><div id="txt-up-laser">🎯 2. Laser (Lvl 1)</div><button class="shop-btn" id="btn-up-laser" onclick="upgradeLaser()">Lvl 2: 400zł</button></div>
        <div class="shop-item"><div>☕ 3. Kawa (Regeneracja Energii)</div><button class="shop-btn" id="btn-up-3" onclick="kupUlepszenieJednorazowe(3, 300, 'kawa')">300 zł</button></div>
        <div class="shop-item"><div>🍵 4. Melisa (Cierpliwość)</div><button class="shop-btn" id="btn-up-4" onclick="kupUlepszenieJednorazowe(4, 350, 'melisa')">350 zł</button></div>
        <div class="shop-item"><div>👟 5. Wygodne buty (+10%)</div><button class="shop-btn" id="btn-up-5" onclick="kupUlepszenieJednorazowe(5, 500, 'buty')">500 zł</button></div>
        <div class="shop-item"><div>💡 6. Lepsze oświetlenie</div><button class="shop-btn" id="btn-up-6" onclick="kupUlepszenieJednorazowe(6, 800, 'oswietlenie')">800 zł</button></div>
        <div class="shop-item"><div>🤖 7. Auto-Taśma</div><button class="shop-btn" id="btn-up-7" style="background:#ffec27" onclick="kupUlepszenieJednorazowe(7, 1200, 'autoTasma')">1200 zł</button></div>
        <div class="shop-item"><div>🖼️ 8. Plakat motywacyjny (+1zł)</div><button class="shop-btn" id="btn-up-8" onclick="kupUlepszenieJednorazowe(8, 1500, 'plakat')">1500 zł</button></div>
        <div class="shop-item"><div>⚡ 9. Szybkie Ręce (Zrzut)</div><button class="shop-btn" id="btn-up-9" style="background:#ffec27" onclick="kupUlepszenieJednorazowe(9, 1800, 'szybkieRece')">1800 zł</button></div>
        <div class="shop-item"><div>📱 10. Skaner kodów QR</div><button class="shop-btn" id="btn-up-10" onclick="kupUlepszenieJednorazowe(10, 2500, 'qrSkaner')">2500 zł</button></div>
        <div class="shop-item"><div>📻 11. Głośnik (Regeneracja)</div><button class="shop-btn" id="btn-up-11" onclick="kupUlepszenieJednorazowe(11, 3200, 'glosnik')">3200 zł</button></div>
        <div class="shop-item"><div>🛡️ 12. Złote Combo</div><button class="shop-btn" id="btn-up-12" style="background:#ff9f77" onclick="kupUlepszenieJednorazowe(12, 3800, 'zloteCombo')">3800 zł</button></div>
        <div class="shop-item"><div>👑 13. Monopol (+50% zysku)</div><button class="shop-btn" id="btn-up-13" style="background:#ff77a8" onclick="kupUlepszenieJednorazowe(13, 4500, 'monopol')">4500 zł</button></div>
        <div class="shop-item"><div>🖥️ 14. Dotykowy POS</div><button class="shop-btn" id="btn-up-14" style="background:#ff9f77" onclick="kupUlepszenieJednoravowe(14, 5500, 'dotykowyPos')">5500 zł</button></div>
        <div class="shop-item"><div>💳 15. Karta VIP (+Napiwek)</div><button class="shop-btn" id="btn-up-15" style="background:#ff77a8" onclick="kupUlepszenieJednorazowe(15, 6500, 'kartaVip')">6500 zł</button></div>
        <div class="shop-item"><div>🧲 16. Magnes na bilon</div><button class="shop-btn" id="btn-up-16" onclick="kupUlepszenieJednorazowe(16, 7500, 'magnes')">7500 zł</button></div>
        <div class="shop-item"><div>❄️ 17. Klimatyzacja (Zamroź)</div><button class="shop-btn" id="btn-up-17" onclick="kupUlepszenieJednorazowe(17, 8500, 'klima')">8500 zł</button></div>
        <div class="shop-item"><div>🤖 18. AI Helper</div><button class="shop-btn" id="btn-up-18" style="background:#ff004d; color:#fff;" onclick="kupUlepszenieJednorazowe(18, 10000, 'aiHelper')">10000 zł</button></div>
        <div class="shop-item"><div>👥 19. Klon pracownika</div><button class="shop-btn" id="btn-up-19" style="background:#ff004d; color:#fff;" onclick="kupUlepszenieJednorazowe(19, 15000, 'klon')">15000 zł</button></div>
        <div class="shop-item"><div>🏢 20. Zgoda Prezesa (x2)</div><button class="shop-btn" id="btn-up-20" style="background:#a020f0; color:#fff;" onclick="kupUlepszenieJednorazowe(20, 25000, 'prezes')">25000 zł</button></div>
    </div>

    <div id="content-custom" class="shop-content" style="display: none;">
        <div class="shop-item"><div>👾 Styl Neon-Retro (Fiolet)</div><button class="shop-btn" id="btn-skin-neon" onclick="kupStyl('neon', 600)">600 zł</button></div>
        <div class="shop-item"><div>🕶️ Styl Cyberpunk (Żółć/Czerń)</div><button class="shop-btn" id="btn-skin-cyber" onclick="kupStyl('cyber', 1500)">1500 zł</button></div>
        <div class="shop-item"><div>👑 Styl Ekskluzywny (Złoto/Marmur)</div><button class="shop-btn" id="btn-skin-gold" onclick="kupStyl('gold', 4000)">4000 zł</button></div>
        <div class="shop-item"><div>🏪 Prestiż: Większy sklep (+10% ceny)</div><button class="shop-btn" id="btn-prestiz-up" onclick="podniesPrestiz(2000)">Zwiększ: 2000zł</button></div>
    </div>

    <div id="content-tasks" class="shop-content" style="display: none;">
        <div id="task-alert-box">🚨 WYMAGANY TASK OD MANAGERA!</div>
        <div style="font-size:12px; margin-bottom:10px; color:#ab9b8e; text-align:center;">
            Co 10 obsłużonych klientów korporacja narzuca zadania zaplecza.
        </div>
        <div id="task-display-area" style="background:#1a1c2c; padding:15px; border:2px dashed #ff77a8; text-align:center;">
            <span style="color:#ab9b8e;">Brak aktywnych zadań. Obsługuj klientów dalej!</span>
        </div>
    </div>

    <div id="content-stats" class="shop-content" style="display: none;">
        <div style="font-size:12px; margin-bottom:10px; color:#ab9b8e; text-align:center;">Twoje globalne postępy w korpo:</div>
        <div class="shop-item"><div>🛒 Obsłużeni klienci ogółem:</div><div id="stat-total-clients" style="color:#ffec27; font-weight:bold;">0</div></div>
        <div class="shop-item"><div>💰 Aktualny stan konta:</div><div id="stat-total-earnings" style="color:#00e436; font-weight:bold;">0.00 zł</div></div>
        <div class="shop-item"><div>📆 Bieżący dzień roboczy:</div><div id="stat-current-day" style="color:#29adff; font-weight:bold;">Dzień 1</div></div>
        <div class="shop-item"><div>🏆 Poziom prestiżu sklepu:</div><div id="stat-prestige-lvl" style="color:#ffa300; font-weight:bold;">Poziom 1</div></div>
        <div class="shop-item"><div>👔 Stanowisko korporacyjne:</div><div id="stat-current-rank" style="color:#a020f0; font-weight:bold;">Młodszy Kasjer</div></div>
    </div>

    <div id="content-career" class="shop-content" style="display: none;">
        <div style="font-size:12px; margin-bottom:10px; color:#ab9b8e; text-align:center;">Wspinaj się po drabinie awansu! Nowe stanowiska zwiększają zysk!</div>
        <div class="shop-item"><div>👔 Młodszy Kasjer (Start)</div><button class="shop-btn" disabled style="background:#5f574f">AKTYWNE</button></div>
        <div class="shop-item"><div>👔 Starszy Kasjer (Zysk x1.15)</div><button class="shop-btn" id="btn-career-senior" onclick="kupAwans('senior', 1200)">Awans: 1200zł</button></div>
        <div class="shop-item"><div>👔 Kierownik Zmiany (Zysk x1.35)</div><button class="shop-btn" id="btn-career-kierownik" onclick="kupAwans('kierownik', 3500)">Awans: 3500zł</button></div>
        <div class="shop-item"><div>👔 Dyrektor Regionalny (Zysk x1.60)</div><button class="shop-btn" id="btn-career-dyrektor" onclick="kupAwans('dyrektor', 8500)">Awans: 8500zł</button></div>
    </div>
</div>

<div id="dymek">🙂 Witamy w Pixel-Markecie!</div>
<div id="controls-hint">
    [SPACJA] - taśma | Cyfry i [ENTER] - wydawanie reszty<br>
    ⚠️ <b>ZATARTY KOD:</b> Wpisz cenę produktu i kliknij <b>[E]</b> | 🚨 <b>AWARIA KASY:</b> Klikaj szybko <b>[R]</b>!<br>
    🪪 <b>KONTROLA WIEKU:</b> Kliknij <b>[Y]</b> (Zezwól) lub <b>[N]</b> (Odmów) | 🔤 <b>APLIKACJA:</b> Przepisz kod i kliknij <b>[P]</b><br>
    🦹 <b>ZŁODZIEJ SKLEPOWY:</b> Reaguj i wciśnij <b>[G]</b> | 🧹 <b>BRUDNA PODŁOGA:</b> <b>[C]</b><br>💻 <b>[A]</b> — obróć się do laptopa (Czarny Rynek) | <b>[D]</b> — wróć do kasy
</div>

<div id="game-over-screen" class="modal-screen">
    ⚠️ GAME OVER / ZWOLNIENIE!
    <div id="game-over-reason" style="font-size:18px; color: white; margin-top:15px; max-width:500px;">Zbyt wiele pomyłek finansowych!</div>
    <button class="action-btn" onclick="location.reload()">Zagraj od nowa</button>
</div>

<div id="day-clear-screen" class="modal-screen">
    🎉 POZIOM UKOŃCZONY!
    <div id="day-summary-stats" class="stats-box" style="color:white; font-size:20px; margin: 20px 0;"></div>
    <button class="action-btn" style="background: #ffec27" onclick="nastepnyDzien()">Kolejny Dzień</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>

<script>
let PALETA = {
    tlo: "#e9dca3", podloga: "#b3c0c8", fuga: "#7e8d99",     
    regal: "#6e4020", lada: "#3a4466", blat: "#1a1c2c"      
};

let motywySklepu = {
    standard: { tlo: "#e9dca3", podloga: "#b3c0c8", fuga: "#7e8d99", regal: "#6e4020", lada: "#3a4466", blat: "#1a1c2c" },
    neon: { tlo: "#1a0b2e", podloga: "#ff00ff", fuga: "#00ffff", regal: "#2d004d", lada: "#050515", blat: "#0a0a28" },
    cyber: { tlo: "#0a0f1d", podloga: "#ffec27", fuga: "#1a1a1a", regal: "#00e436", lada: "#ff004d", blat: "#000000" },
    gold: { tlo: "#f5f5f0", podloga: "#ffffff", fuga: "#ffa300", regal: "#4a3c31", lada: "#d4af37", blat: "#1c1c1c" }
};

let zakupioneStyle = ["standard"];
let aktualnyStyl = "standard";

let produkty = [
    ["Chipsy", 7, "pack", "#ff004d", false], ["Sok (18+)", 5, "can", "#29adff", true], 
    ["Ciastka", 6, "box", "#ff77a8", false], ["Woda", 4, "can", "#29adff", false], 
    ["Ser", 12, "cheese", "#ffec27", false], ["Przekąska (18+)", 15, "box", "#ff77a8", true], 
    ["Chleb", 4, "bread", "#ffa300", false], ["Masło", 7, "butter", "#fff1e8", false]
];

let aktualneCeny = {}; 

let wynik = 0, zarobek = 0, combo = 1, cena = 0, dal = 0, reszta = 0, wpis = "";
let cubes = []; let kolejkaKlientow = [];
let blink = true, graAktywna = true, tasmaPracuje = false;

let trybGry = "normalny"; let maxGwiazdki = 5;
let gwiazdki = 5; let cierpliwość = 100; let cierpliwośćTimer = null; let czasKlienta = 0;
let typPlatnosci = "gotowka"; let aktualnyDzien = 1; let klienciWDniu = 0; let utargWDniu = 0;
const KLIENTOW_NA_DZIEN = 10;

let stanKasyAwaria = false;
let awariaKlikniecia = 0;
let produktZablokowany = null; 
let aktywneKredyty = 0;
let spieszySie = false;

let wymagaWeryfikacjiWieku = false;
let wiekKlienta = 20;
let zweryfikowanoWiek = false;

let stanBhpSzalenstwo = false;
let bhpMnoznikPredkosci = 1.0; 

let energiaPracownika = 100;
let prestizPoziom = 1;
let mnoznikPrestizu = 1.0;

// SYSTEM KARIERY
let aktualnaRanga = "junior";
let mnoznikKariery = 1.0;

// SYSTEM TASKÓW ZAPLECZA
let licznikDoTaska = 0;
let aktywneZadanieZaplecza = false;
let kliknieciaTaska = 0;
let wymaganeKliknieciaTaska = 0;
let nazwaAktualnegoTaska = "";

const pulaZadanKorpo = [
    { nazwa: "📦 Rozkładanie kartonów w magazynie", klikniecia: 8 },
    { nazwa: "☕ Czyszczenie i odkamienianie ekspresu", klikniecia: 6 },
    { nazwa: "📋 Liczenie i weryfikacja stanów magazynowych", klikniecia: 10 },
    { nazwa: "🗑️ Opróżnianie przepełnionych koszy korporacyjnych", klikniecia: 7 },
    { nazwa: "🥛 Rotacja towarów na półkach (zasada FIFO)", klikniecia: 9 }
];

function switchTab(tab) {
    document.getElementById('content-upgrades').style.display = (tab === 'upgrades') ? 'block' : 'none';
    document.getElementById('content-custom').style.display = (tab === 'custom') ? 'block' : 'none';
    document.getElementById('content-tasks').style.display = (tab === 'tasks') ? 'block' : 'none';
    document.getElementById('content-stats').style.display = (tab === 'stats') ? 'block' : 'none';
    document.getElementById('content-career').style.display = (tab === 'career') ? 'block' : 'none';
    
    document.getElementById('tab-upgrades').classList.toggle('active', tab === 'upgrades');
    document.getElementById('tab-custom').classList.toggle('active', tab === 'custom');
    document.getElementById('tab-tasks').classList.toggle('active', tab === 'tasks');
    document.getElementById('tab-stats').classList.toggle('active', tab === 'stats');
    document.getElementById('tab-career').classList.toggle('active', tab === 'career');

    if(tab === 'stats') {
        aktualizujStatystykiUI();
    }
}

function aktualizujStatystykiUI() {
    document.getElementById("stat-total-clients").innerText = wynik;
    document.getElementById("stat-total-earnings").innerText = zarobek.toFixed(2) + " zł";
    document.getElementById("stat-current-day").innerText = "Dzień " + aktualnyDzien;
    document.getElementById("stat-prestige-lvl").innerText = "Poziom " + prestizPoziom;
    
    let nazwyRang = { junior: "Młodszy Kasjer", senior: "Starszy Kasjer", kierownik: "Kierownik Zmiany", dyrektor: "Dyrektor Regionalny" };
    document.getElementById("stat-current-rank").innerText = nazwyRang[aktualnaRanga];
}

function kupAwans(ranga, koszt) {
    if(zarobek < koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    zarobek -= koszt;
    aktualnaRanga = ranga;
    
    if(ranga === 'senior') { mnoznikKariery = 1.15; }
    else if(ranga === 'kierownik') { mnoznikKariery = 1.35; }
    else if(ranga === 'dyrektor') { mnoznikKariery = 1.60; }
    
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    playSound(900, 0.3, 'sine');
    
    let btn = document.getElementById("btn-career-" + ranga);
    if(btn) { btn.innerText = "AKTYWNE"; btn.disabled = true; btn.style.background = "#5f574f"; }
    
    document.getElementById("dymek").innerHTML = `👔 Gratulacje! Awansowałeś na wyższe stanowisko! Twój mnożnik zysku rośnie!`;
}

let wymagaAplikacji = false;
let kodAplikacjiWymagany = "";
let kodAplikacjiWpisany = "";
let zatwierdzonoAplikacje = false;

let uciekayacyZlodziej = false;
let zlodziejZlapany = false;
let zlodziejCzasNaReakcje = 0;

let wymagaSprzatania = false;
let jestZlotyKlient = false;

let ulepszenia = {
    melisa: false, tasmaLvl: 1, laserLvl: 1, kawa: false, autoTasma: false, aiHelper: false,
    zloteCombo: false, monopol: false, szybkieRece: false, kartaVip: false, buty: false,
    oswietlenie: false, plakat: false, qrSkaner: false, glosnik: false, dotykowyPos: false,
    magnes: false, klima: false, klon: false, prezes: false,
    roslinka: false, neon: false
};

const kosztyTasma = { 2: 200, 3: 4000, 4: 6000 };
const kosztyLaser = { 2: 400, 3: 2200, 4: 5000 };

function przeliczInflacje() {
    produkty.forEach(p => {
        let zmiana = (Math.random() * 4 - 2); 
        let nowaCena = Math.max(2, Math.round(p[1] + zmiana));
        let nowaCenaFinal = Number(nowaCena.toFixed(2));
        aktualneCeny[p[0]] = nowaCenaFinal;
    });
}
przeliczInflacje(); 

function playSound(freq, duration, type='sine') {
    if(!graAktywna) return;
    try {
        let audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        let oscillator = audioCtx.createOscillator();
        let gainNode = audioCtx.createGain();
        oscillator.type = type;
        oscillator.frequency.setValueAtTime(freq, audioCtx.currentTime); 
        gainNode.gain.setValueAtTime(0.04, audioCtx.currentTime); 
        oscillator.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        oscillator.start(); oscillator.stop(audioCtx.currentTime + duration); 
    } catch(e){}
}

function generujPixelTexture(typ, kolorBazowy) {
    let canvas = document.createElement("canvas"); canvas.width = 16; canvas.height = 16; 
    let ctx = canvas.getContext("2d"); ctx.imageSmoothingEnabled = false;
    ctx.fillStyle = kolorBazowy; ctx.fillRect(0, 0, 16, 16);
    ctx.fillStyle = "rgba(0,0,0,0.15)"; ctx.fillRect(0, 14, 16, 2);
    
    if(typ === "can") { ctx.fillStyle = "#fff"; ctx.fillRect(0, 6, 16, 4); }
    else if(typ === "pack") { ctx.fillStyle = "#ffec27"; ctx.fillRect(4, 4, 8, 8); }
    else if(typ === "cheese") { ctx.fillStyle = "#ffa300"; ctx.fillRect(2, 2, 4, 4); }
    else if(typ === "bread") { ctx.fillStyle = "#7e2553"; ctx.fillRect(4, 0, 2, 16); }
    else if(typ === "podloga") { ctx.fillStyle = PALETA.fuga; ctx.fillRect(0, 15, 16, 1); ctx.fillRect(15, 0, 1, 16); }
    else if(typ === "liscie") { ctx.fillStyle = "#008751"; ctx.fillRect(2,2,12,12); ctx.fillStyle = "#00e436"; ctx.fillRect(4,4,4,4); }
    
    let tex = new THREE.CanvasTexture(canvas);
    tex.magFilter = THREE.NearestFilter; tex.minFilter = THREE.NearestFilter;
    return new THREE.MeshStandardMaterial({ map: tex, roughness: 1.0 });
}

// === LOGO SKLEPU EARL! ===
const LOGO_EARL_DATA_URL = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAACmCAYAAAA1QFEhAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAGYktHRAD/AP8A/6C9p5MAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAAHdElNRQfqBRUPLSYtVyK2AAAAJXRFWHRkYXRlOmNyZWF0ZQAyMDI2LTA1LTIxVDE1OjQ1OjI2KzAwOjAwzZZoegAAACV0RVh0ZGF0ZTptb2RpZnkAMjAyNi0wNS0yMVQxNTo0NToyNiswMDowMLzL0MYAAAAodEVYdGRhdGU6dGltZXN0YW1wADIwMjYtMDUtMjFUMTU6NDU6MzgrMDA6MDB3S4raAABQi0lEQVR42u29d3gk5ZXv/6nQOSin0WRNznmAATMYk2wwyQQTbNZcx7sL9q7t+/N6HXYNa+8a24B3vfau75prwmIDBhswGDAwYNIAA8yMNEmTNKOc1TlU1e+PqtK0ultSt9RStzT6Po8Y1KqueuutOuc97wnfI2iapjGDGczgtISY7wHMYAYzyB9mFMAMZnAaY0YBzGAGpzHkfA9gBmODqqr4/X4URUEQBPyBAM0nT5Lo0hEEgWPHj3PgwAEEQcj43JqmsWTxYhYsWDB4Pg2wWizMmzcPWZbRNA2r1YrL5cr3VMxgHJhRAAWIaDRKJBKhs7MTVVVpbW2lpbWVgYF+9uytR4nHCUciNDTUEwqFEQSBUCBAe3MzyS5dORJDiUeyHoMmW8FmTfwE2WJl1tw5SIYCKC8rY/GSJQgCeNweVq9ejd1mY+HChXi9Xmw2G+Xl5ciyjMViyfe0ziANhJkoQP4QCoUIBAI0NTXR2dlJfX09vX197Nm7F5/Px9EjR4jH44R9fsI+H5Kq4VQVBHTNXY0FCwIa4ESkBgvmOq8BAlCChH0MO70BFPyoJNoNMTSaiRI3zh1ApZM4GqAgEJQkNFHAXVqCZLPidruZP38+xcUlrFq5kpKSElauWEFlZSXV1dV4PB5keWYNyicmXAGoqoamgSRlboJON6iqSl9fH/39/Rw9doyDBw6wt76eQ4cO0dzcTGdbO6FgAC0Ywg4UI+JAYDZWZKAYiRJkLIAHCQFdAK0IQwQ03QyP9eEO97QSz6egKwWAOBo+VBQ02ogTRiWISjMxImj0ohEXRUSnA29xMaXlZSxZsoS5c+eyYf0GFtXVUVNTQ2lpKW63O9+P7LTBjAKYAASDQbq7u2lsbGRvfT176+t5//33aWttwdfVQywQxIlKJTJFSNRgwYFIGRJORFyIiICELojmA9IYu0BPNIQ0/6rGTxyNACoDxk8fcbqMn15UwqKEs6gId0kxC+vqWL9+PcuXLmX16tXMnz+foqIirFbrmMY1g5ExswXIAcLhMCdOnGBv/V527nyb995/j+PHm2g9eRJlwI8bjRpkZmGhCgtuRIoQsSMiwxCzPfHf6QTTajHvT0O3HoJo+FDoRaGDOC3E6EFlQBJxlpZQO2c2i+oWsXHjRjZv2sSyZcuorq5GkqR839K0wIwCGANMx9x777/Pzp07eWvnW+zds5f+zi4s0ShliMzBYgi8jAcJBwLmKzudBT1bJCsGBQij0UWcTuI0EaWFOH1o4HJSNWsW6zds4IwtW9iyZQsrDd/CDMaGGQWQIXw+H01NTbz55pu8+dZbvPnmmxxvbEQKhahAZC42Zg8KvDi4P58R9uyRqBRUdIXQh0IrcU4Q5SRRegFLcRHLV69i69atbDvzLNavX8+sWbOw2Wz5voUpgxkFMAICgQB79+7l+Rde4IU//5nGgwfpa++gNB5nLlbqsFGdRuBnJjS3SPYrhNHoR+EEMY4Q4QRxwg4HlbNqWL12LRdfdCHbz93OwoULZ8KPo2BGASRhYGCAffv28fKOHfzpuef4YNcuor29zEJiCTbmYqUSGceMwOcNyQrBh0obcY4Q4RARegSBktpZnHnWWVx0wQVs27aNBQsWYLfb8z30gsOMAgAikQiHDx/mqaef5tk//Yk9H3xAtKubGkSWYmcBVsqRsBmv3ozQFxbMLYMGBI0wZCMRDhGmR5IoqZ3F5i2bueLjl7N9+3Zqa2sRxZkseDiNFYCmabS2tvLGG29w/4MP8u7bbzPQ0ka1qrHCEPpSJKwJQj+DwkeiMgih0U6cQ0TYT5gBm42aBfM566yzuOrKKzn77LMpLi7O95DzO1+nmwIIh8M0NDTwyKOP8sTvf09L42GKolGWYWcldiqQBrPrTquJmYZIVAZhNJqJsZswh4kQdblYsW4dN95wAxdfdBELFiw4La2C00YBdHd38+yzz/KHp55kx8s7iLR1sAyZlTiYjQXnzEo/7SGihxl9qBwmyl5CnJQFKhfM56KLL+byj3+cs88++7TyFUx7BXD06FEeeeQRHn/iCfa+uwtvNMoaHKzARgUyEjN7+tMNpmUQRaMDhfcIsZcwQnExZ33oHK675lo++tGPUlpamu+hTvxcTEcFEI/HOXLkCI88+ggPPPAgx/ftZw4iW3GxAAsuozhm2t34DLKGaRX0o9BAhPcJ0mOxsP6sM7n1r/6Kiy66iOrq6nwPc8Iw7RTAgQMHuPvuu/nT88/RfeQ4yzWJ9TipMUJ3M6v9DNLBDC2G0DhKlLcI0myTWbhiGVdcfgU33XgjixYtyvcwc3/f00UBHD58mId/8xvuu+8+2g41shwLW3FRa5j56rivoCGgoc2QKE1rmNuDCBqHiPImAY6hMmdRHbfddhs33nADZWVl+R5m7u53qiuAQ4cO8T8PP8xDDz1E6/6DLENmM05qsSCTC8EH0IjYygi4ZlPctw9RjeX7toeMq6VmOzVtO7CHu0hXyCug0lOyCr97HrNPPoegKfke+JSAiB49aCTKToKctIisO/MM/uqWW7js0kupqKjI9xDHjSnLxtDR0cHPf/5zHnjwQdoONrISmQspogZ5UPAzEf7kTYFmrAECKqdoNSDkqKSjcise32EkNZr2O/r/qcZZE89z6rhT1xMh5dqicbWh5xg63qHjilqLaJl1HuXd7+MMt6MhGlaKNngtAQ2fZwHtlWcw++TzCKhDxjfcPZ86T/rxpJ7n1HWng6WkonMurMLGIqwciUV585XXuf3Nt/jlpl/yxS98gWuuuWZKRw2mnALw+Xw8/vjj/OK//pNdr73Bck3m0xRRizwoUtms+j7XXBTZgSrIqJKVov6DSEoYn2seIWc1lpifov6DOEKdVHS+jaREiMlu+oqWIGgKohbHFunFGWwh5KjC556PqMYpGjiEJeanz7sEBBFFsuHxHSfoqCJiL8MZbEETRILOGgRNQ9BUvAONxGUHAedsZCVIUX8jkhIaHKsmSAx4FhGyV2CL9FA00AjGd0OOSsL2Uuzhbjy+Y8RlJwPeOuKSg6KBgwiqgipaCToqidpKcAWascQG6CtahiPUji3Sy4CnjpC9AmtsAE2QKOo/RMRajN8zD0FTKOo/hCU2AAioopX+osW4/U0IWpwBTx1e32EUyU7YVo7XdxhBy439lW+YimAFNuqw0hiN8sbrb/G/33+fJ59+mi987nOce+65U7JEWfrud7/73XwPIhNomsarr77KN/7+7/npj3+M5UgTF+FmG07KGNvEC2gcn385R+dfhSrZODn7YjRRwhVopmX2BQQdVZycfRGyEiZm8XBkwSco736fIwuupqPyDKK2Eo7NvxI5HkRWguxf9jnCjkp6S1fRV7ycov4DHFp8M60129FEK0FHNSfmXgoGQacqWun3LqG15ly6yjdQ1rObzorNBF21NM86H1WSKe4/gLnqttSez7H5VxKzuGmr+RCaIGIPd9Na+2HiFhdBZy3NtRfgDhyntfpDtFWfDaKEJRbQLZiqM3CEuzg2/wqcoVbs4S4aVvxvHOEuwvYK9i+7lbjFQ0flGbRXnUVl5046K7bg98ynvfJMgq5aynp2I6CiiDYOLP0MshImLrvZs+ZvKe4/SF/xcrrLN1De9a5hgUwfaICEQBUyy7Djiim8Uf8+jz37DC1tbSyqq5tyocMpYaedOHGCr33961x73XW89tjvuDgscwMlrMSGZYiRnT1UQcIR6mDJwfuo6nid3uLliJqCd6ARW7QPVZTpK1qMJohogkTIUUlvyUoWNz7A0gP/F7f/BJoo0155FpaYn5UN/87Sg/+Nz7MAv2c+miBR0rePJQfvwxL3E7O4cAZbKO3ZTWXnW8xqfQlRi1N35Le4/cfx+o5ii/QA0O9dgibodk1cdhp7/VdY1fBTZrW8SHvVNhTZgaDGqW1+gRX7foY93El36RrdqhGtFPftR44HaK8+G1egmdLePWiCZfDRa4KEKsi0V51Fcf9BVtb/lJrWHZhbGo/vKPZwJ5Iaoa9oGYqkm7uyEsTjO0a/dzH9RYuxhXvo9y7C512Id+AQohbP92szYVABBwJn4OBWSlnf4ef//uRuLr/iCn72H/+Bz+fL9xAzRkErgEAgwKOPPso1117Lz+76EQvaermFErbgwDpOwT8FAWt0AFkNYon50QSJ7tLVHJt3BZaoD1ukl0TKClWQ0QQRUY0OWeE0UUbQVARNQVAVzL29oKnYw12IWoyKzp3UHXmY9sozObzweiLWYo4s+AQlPXupbn+VAe8ijiy4BkkJYw93D44PMBSQjKDFEVARtaHXEJUIaBoIAqKmMLfpKarb/sLhuuvpqNhKSc9u5HgIKR7S70OUUUWbLtCCgCLZEdUYAsrgvYYclTQuuhFVtGAPdZLorxDQKOnbx4B3EQOehcxq+TN9xcsJOSop6j+U71dnwmF6b9yInIuLmyhCqj/A//nKV7ju+ut5eccOVLXwt0AF6wM4dOgQ3/ve9/jdY49RFYxwI14WGCSZOZ9WwfyPIWyihbjsJOisJmIrwRlsHTzQEe7EFWjm4OJbcAdOEHDPobR3LxWdb3Ng6WdoWP4lYlYPHt9RPL5juuAa5+4s30zAVYukRtFEiRNzPkpX+SZq2nZwZP7VyEqIuGwn5KgibC/DEeoYvK4lFqCq43Waay9gwFOH3z2XmrZXkeIhFMlO09zLkJUQEWsxpT17aK0+h7jFjajGkeNBrNF+BryLsMT9ePxHOT73UnpKVxOxlSKqMcq73uHIwmtRRCtBZw2gWzxx2UHYXk7IUUmiA1BDwO0/RszixhLzUdH1Ls21H8Ee6cYRap8WTsBMYKrEBViooZiGSIRX/vgM1+/axRe/+EW+9MUvFnS0oODCgLFYjD/+8Y/84z/9Ewd27WIbTs7AgQcx94IPDHgWoooWSvr34XfNIWItwes7TF/xclRBRlKjWGI+As5aTsz9KOvfuwNFstNftBRQOT73MmraXmXuiafxuefj8yxAioco6duHJeant2QF1ugArkATQWct/d5FSGqEov5G/K5aIvYy3ZGHSlF/I2F7GTHZhSUeRFSjFPcdwHzNVNFCX9FSwo4KHME2ivsPokh2+oqXISoRIvZSPL7juP1NDHjr8LtmY4v2Udy3j4itlLC9nNKevYRtpfSVrMBiOPvcvuNYYwP0FS8nZvEw4F1Ib8lK1r33z4Sc1brTMdqPKkqU9tQjGOa9Joj0lqxCjgfx+I7QU7IKWQmdFhbAcBCAPlRexs8HYpyt553Ld7/9Hc4+++yCLDYqKAVw7Ngx7r7nHn79q19R3O/nI3ioM7juJ2qQ5kZCD8Hphp2WEL7TkGmedT6ts87F23+IuiO/obn2I8Rkt75SOypYvu/nuAMnSQzr6SugMCSEJiTEKBKvdwqJocHEUFzyeDXjnEPDdImfnzoONKSEe0v3HegtXkFnxRYETaW7dC3V7X9h3vE/DBMaHG7+1LTHnG4Q0NOLG4jwLAOIVVV85Stf4Utf/CJerzffwxs61kJRAH/5y1/48t9+hYNv7+IM7JyBAzdiAfiRBXpKVxG1FlHWvRtLzEdf8VICrtl6uK//QMIWYapCI2IrpadkNapkwxFqo7hvfwElPE1NCEA7Ci/h56BV4GNXXs4d37uDxYsX53top8aYbwUQiUR44IEH+N73vody/AQfxcvCCV71s54kYxUcuqqbozP3+FMdWoJbVTjtV/FcQUCnP3+PMM8TYN6a1fzLD37ARRddVBBbgrwqgO7ubr77j//IA//9KxYFonwED6UTtNefwQzyBXMxayLGM/jwlRXz7e98h8985jN5b66aNwVQX1/P177+dV599jnOU22Dob1CWfVnMINcQwAGUHkOP/V2gZs+/Wl+8P3v57WvQV4UwFNPP823vv1tTuzaxccpYgnWQffXiIPN4JgZzKCQYVYa7iTES2KEC6/8OHf80/dYsWJFfsYzmQpAURQeffRRvvKVr2BvbecSvMzDMqJQm7vrMBoqYEfISFnMYAaFCpPO/H3CPIePJWecwc/+/d/ZsGHD5I9lshRAOBzmX/7lX7jnnnuo7vVzJV6K0nj5E+vNYmh0o3CQGPVEiKJRh5XtOHAVRIRgBjMYOwTgKDGeYAD34oX84Ps/4Oqrr57cMUyGAggGg9z5z//M3f/6QzbG9NRJT4IAJ7K39qPSiUI7cU4Qp5EoviS34KW4OQfHjAKYwZSHAHSh8CQD9FWXc88993DttddO2vUnPBU4FApxx513cs+PfsSZMYnzcA3SboOe+uJD5SRxmohxgCidKERGEO924jPCP8WR3BH5dIUGVCBxFUU83tbFbbffjqZpXHfddZNy/QlVAOFwWBf+u+7irKjEebgHL2g6QxqI8gYhWogTy+B1EIE5BZYnMIPsoAA9KGhAKSLytMijGDtUoBiRqyni8bZubr/9dkRR5Jprrpnwa0+YAggGg9xxxx3cc5e+8m/HhQVdaP2o7CPKB0Q4RiwjwRcADyKbsLOW6dn9NbHXYLYMB1OF7DSOxosEeYcwGrAWGxfgGmy7lg9kko4zUfNrOrQDhpN7O27+1N7NbbfdhqIoXH/99RN67xOiAEKhEN+74w7u/dGPOSsmcx4uZKADhQ+IcIAozcSNNWDkySlCYh4yC7EyG5nqBC7/6QQzUeR9IoTRqEEe1c4REKgyehY6EfEgFvTciOhOr9cIDW7x3iLMYqwsx5qXBLAYGoeJ0YsyRAVpgB2RSiQk9PfQgj7nlqTjRotiDafaQsa1TxLjCDH8qAhAkduDXdH4ype/DDChSiDnCiAYDHLHnXdy710/YltMZjsu+lB4nwi7CNPNyISUVgRqkJmPhbnI1CBTjIScMNmF8IInPtjxjkkA9hPlMXwpDs/RYDPCom5EZiFzBg7mYynIUGkMeJvwEP9ODI1AnnI/NeAVQrxMMK0VKqK/jyLC4FbFhUgVEk5EapEpQ8KNOESQzDc8ikYfypBnqqEvhAHD73WEWIpHqzPQz6qIE0cswpe//GVEUZwwx2DOFcC//+xn3PPDuzg7JrMRB68T5C3CdI0i+OVIrMHGPCzMNVp1JZrEhZQeLKA7Lo8QQwEWYqF4HGFJFdhFOGvhBwaFKYRCJwpHiLEaG1uxU41cMErAzIlPfg9sCLjzUHdgPsP3CA+7BVXB2KhoBBOeTX3C2IsQqcPKLEOUVKCNOBE0BlDpJE4goVpEg9Fd2JpGKBblarw83d7Dbbfdhs1m4/LLL8/5PORUAfzmN7/hR3fdRV0MHIg8xAAniI34WjsRWYWVbTioSiD2LJSVPhkC0IvCH/BzgCgaOhnEtXjHpQQsOdoD+1B5nRCHiHIuTtYZtGmFMJfpnmkNMnNGSQabKCiQkf9pOESM1mIdhFK2D+OFhr7tuAIvD7V38a3vfIf58+ezdu3anM5BzlTvjh07+PKXv4ykaQQdNp7Cz/FhhF9GoByJzdj5NF6uwEN1ggYthJd1OKjoZmMDURTj98PEOEJ0zCIsAbVpdLGIrhjS/YzmJOxE4XH8vECQSAHUK5qKsz/pjbAiDG7v8jWuXCDXi5bXoIgvQuSjeGn/YDef+9znaGxszOn958QCqK+v52tf+xptbW3YbDYisWja4yRgHhY2YWchVjyIWCh8oTchAHuJ8D6RlL/1o475HtI5igTgTBwsw5r2vEE0uogTROMYMdqIpyhbBY1XCQLwEZx5D7fF0VIcvw5jj50vFOp7V5nQ32IuMldRxEM7d/Ktb3+bn957L+Xl5Tm5zrgVQHd3N//0T//E22+/Dej1/elQgcR2nCzHOqQ5ZyHt7UeCgC7kLxIcsh800Y4yyLGTC1gQWImNJViHVS0aNlT0sGoDUXYT5miS1aUAbxCiEokNFF4DCydi3pgHrOjO0/4CewstRnTHhAbMx8KFePj9w79lUV0d3/nOd5Dl8a/f45p7RVH44Q9/yCOPPDLsMSVInIGDmyliM/bBHP5C1bzDQQAOE6WN9HTXAyiGRyC3UI34cLofk6jLi8iZ2LmJIi7EhT1JDUXQeJYATcQKjuajkSj9RvhrMqEBLkTWFGBOyTws1KWx/DZiZ4tm5d9++lOeeOKJnFxrXO/D008/zS9+8QvSlRPICKzHzqfwcjluqpCmjKmfDhrQksbMNtGLSiAPL7I5NhVwInAuTi7AlWLuD6DyKiFCefIHaIAXieKkV64PlQ7iedsELMWaMqaJgs1mw2q1jnrcYiyDnawTIQLn4qSqP8A3/+Ef+OCDD8Y9pjHf+b59+/jGN75BX19fyt+sCFyEi6twM8t4Faeq4CdiJEPRh0prHl9kONVxcAt2NqbJrdtPlMPE8qYAShBZkbTixtB4l7ARbpv8MVUhswXHpKgAj8czKgOQ6SBP94xMq+ViPHQeOMAdd96J3+8f15jGdN+hUIi77rqLhoaGlL9JCFyAi23TjOEnikbvCLkMMTSax1ikpDG+cFTyuazGM5g3JGdNv0ajEb3IBwT0vWyydbKPaN62JyKwDhslY2wvlw26urro7e0d8ZhKpBHDohowC5ntuHnmD09y/wMPjPv+s8ajjz7Kww8/nPZv640klEJOSc0WAnrxSjMjt7sa6/0qwEly10pL41TdRPJrfYI44TxuA2qRB5NmTETRaMrh/Wc7phIjCa0QUIs8pFR+uDFvwM6iiMKPfvSjcW0FslYATU1N/PCHPyQYDKb8bRYy5+HENo1WftAVQAcK/lHuqhdlTK9xOgvAgoB1nGK6HOtgfoWJECqhPKbeehDZiC3lxQuPI4w6XojoIddZBdAoq8KoPRgNVgQ+jJuexkZ+fPdPiEajGXwr/b1nDFVVuffee9mzZ0/aAW3HSTnStBJ+0Pf+TcRGLV7qR80ZU4EHcVyZhRp6NqYn6RH7UBnIk7MSdGVaiyUlUtE+QVGUTOeqyLCY8hklkRFSFPZIY65G5ixcPPHoY/z+978f0zWzut9XX32VBx98MOVzM2ll5TBJK1MZAnpWXT3RlM+TJy+XUY5cCKiIXmORjHw+I92RJQzmgpiIoeX93VmLjTpG99JPFNwIlGW5gG7EQZk/xN333ktLc3PW18xYAQSDQe69917a2tpS/rYEKx/CkfdMs4mAgL769yS5ziqNSrBEdKPQnVRWmk+IMEn+7ezgRMRZYOMytydbseesLiNbVCGn5ckcacxuBLbj5oPX3uDBhx7K+poZP4VXXnmF559/PuVzNyIfxjmq42KqwjT/k7EGG3OTvOx+VOqJZHjmU9DQcuqZN3vT7SLMO4SSrpV/FMIY0kFFzwtYlSeHYBlS1spHBRZjZZkm8ev77+fEiRNZfT8jBRAKhfj5z3+Oz+dL+dtyrMzGUmDJlLmBgE7a0JKmhHU+FuaneVytxLNyBAqA32A/TsRYhUREV0TPEeBx/PQmPRmJ7NmGTifYEDgbB0V5sFAqhon/jwYZ2IKTY3vq+Z+H/yer72Z0ly+88AIvvPBCyudlSJyFowB8pxMDk7E12fz3IlKORB2WlPixSfaQzYOMoqVEAYoQs8qjMB9kM3Eex88OowowGcuxUYU8LRV2LqCih+LOmeRNisVIABrrmGdjYQkyv/rVfRw7dizj7456j5FIhPvvv59AIJDyt03YmVVApBO5hgrsI5LCWDMXCx7Dw16V9NDCaPizEC8B3XcQSppFL2JG5qBZSdhCnD8T5NcMsHeYbcgabFycplagEFBIIxLQ8+5XT+JWoByJqnHIkgXdCji5/yD/M0yOTjqMqgD27NnDa6+9lvJ5JVJeyTlF42eiXhwBCKByKI33f7ZRqikjpITZQqhZOwJjacpki5FGfDim4PcbOf7/jwGeJ5A2W1FGYCN2Po47KyfTZCKAlrc6hWRo6HUVF+Ma86qcLeYbi8p4WKXmYmGeJvD4E0/Q1dWV0fdGVACKonD//ffT0tIy5HMBWI+d0jzE/E0H12Fi7CJM6wRlkOnZfyrdSat5CRKLjXCniO65TYSKnm03HhNbRKfLTicMZvixzxD8++jnafzDpikXIXIxLq7AXRCOWnPekq0bPyrBPOYnpBtnCRLr0iQt5Ro2BJZiHbeqkYH1ODmwew/PpXHYp8OI99bU1MRTTz2V8rlJ6jGZD8t88SNo7CDI/fTzG3zczwCN42DjGQmtxFOy5uZhGYzVmqW4ya6bE8RGbGwyGtKVSyeu+LuI8BADPI2flhFSjxZj5Sa8nF1gdRlWBCqngCtSALbhZOkE5wbUII/aIzMTaMAirBSHQjzy6KPDcnMkYkT/3Wuvvcbx48dTPl+PnSLECXckmS99HN20biLOTkIcSmBS7UbhPSLUjTtxdiiiaDQkibGAnvMgcSrpZxYyxYhDvPhBNMJo2MchdLrC0+so40AHcQ4S4z3CtI9iYdQgswE767HhNZ5ToQi/eW9TwXFsJi1twcGRcSr1kTAbGWcOFLSe/SmwFDs733yTI0eOsHz58hG/M+xziMfjPPXUUyjKUNPSg8jcCX58IrqA+VBpJk49EVqI0zFMumjcyCLLlQIQ0E3s5O1FCRKzExw1eo27mKIAfKj4UCkZh8kdRqOZGD2oHCTKPiL0jaJyiw2/zBnG9gymDuMSFJYj0IQK1GGhDgsNjC3ffiQ4EVhhlG7nQr0IwCJsvN7SxosvvTh2BXD06FHeeuutlM/nYZmwMJKIvto3EWM3EY4So3OUHHEBqDNW5Vym4bYQT6Hpno2c4veQjfTNwwnJQvFxct2bfPUvESSUJkSYDC8i67CzwQjxFWJPgHT3mIg4GsECHbUNgXNwcpx4znsYLMDKvBzKk4pulc5G4Jlnn+XWz9yK3T48FdywCuC1117j5MmTKZ9XGdVKuXxUpmOviRjvEmFvmtBbOtgQWIUt5+GaOFBPJGUEswzvf+K9S+grbyKiaHSiMLLuHRm9o+QGmv6H+Vg4GydzEohXClOMhiLZbxIxkqEWJWVXFgJMTr5N2NlBcNznS8RCLFgQcqpWbAgswcaud3dx8OBB1qxZM+yxaRWAoijs2LGDeHyoCewwvJW5ZPgxk23eIswuwhnF0G0ILMLKZuwsxJLT8mM9+09NycyzGZVa6e690gjZJY68zcgInCgP8iKsXIyLmoRWaVNB8EGf46o0c1nI4xeBs3BwjBjH06SGjwVexJw4/9JhNlZebOvgrbfeGlEBpH0/29vbef/991M+L0KiKEehPwFdYN4lzK/p5xWCIwq/jEAtMttx8imKuA4PK7BOiHfbh5rCFFuBxJxhEjXKjVZRiWghPqFhrTbivGs4BM3QWiHuoYeDw+j8lIhCHr9JaXZWDoveVmKbkEQ6Fb1UuFRVef3NN1MW8kSktQAaGhrSNiBYhGXciSSm1m8lzjuE2Ul42D2+hG52zzPy7hdiwWX4xk1m3FzDLP9NzsyrRsaZ5t5VoBSJCqQhCiyENi5ugNGsLLMD0D6irMJKHVbmIg+hXC9kaGl+D+aRFCTTMS81GpnuGUPRVyJkBJZgHeT+zzXsCMzGws6dO+no6GDWrFnDjCMNDh06lDb11zEYmMoeiXHs9wjzBuFh97kyArOR2YSdZVhxG9zxk9EyzEzkSRbeCsPMT3dtKwIVSBxNMQ3HtlIIMEhUeYBoSi1CInpReJUQOwlTjcw6bKwywn+FLEz6+zD0berIcW+FXMMMs12Iixbioza6HQnVSRGlXEPnX7TyYns7zc3N2SmAhoaGFKpv2WhWkK0CMAVfb4Uc5VVCHBvBrz0XC+fiYIGx2sPkNRARYLDTTiIsCKOyGycbtGEjFFiagSCmWw2LEdmOkx4UDhI19p5x+oZ56SJoHCfGCWK8Q5gzcbAWW0ElACXeXzkSLgQGEkZXaOMcbuyVSGzEzvMExjzmdZOQS1OOTKi3jyNHjrB58+a0x6QogGAwSH19fcqBboSMihVMgdfQveEDqLQY+9XDxIYVfS8iW7CzGcdg/DwfL0TESElNRAXSiHs1AVJyxcJoGXf7tSIgIQwxgM29fTkSFTjYioMO4uwhwgGiw5KIqugVgU/g5wgxPoQjY5qpyYQNYUoTyGzGzhGiNI7BIehGpG6CG6KaNGeOeJwP9uzmuuuuS3tcypvR1tbG0aNHUw60IuBI88CEhH+jaETQ6EPlBDH2E6UThQHUEWPZs5C5ABfLjQhDvpJX9AaWaorglhpOvpEUQHKEINNXW0Mvq3YgDJmjFuIMoFBiNFQR0TP8agwe+7eNBqWdKGnnNo7GLsI0EeNCXKzFVlArbCGNZSxjL0JkNXYOj4HQfSlWKie4jkZD9wOUItJQ30A0Gk3blCRFAXR3d6etJCpBGsz+T1zlgwZffhtx9hmts/TKrtEdOl5E1mNnm0HAUAihrHbiKU7JyhH2/6cmcuyrmQMBNyIDCYqnF5UOlCGJR+a/xYh8BBdn4uA4cRqIcIIYHWloS7tQeIYAAkxqeetYEENDRRsjLcbkwdzSNo1B+J2IbDRoxyZ6oZMRKEbixMkTRCKRzBRAb29v2rBBJdLgCq+iO5+OEmMfUboMEoxMJ8NueEC34WAulrxkrqXryBtCoz5J/M2Kv7E4P9NdIxkmF90KrLQkmPVmo5F0hSjmONyIrMLKCqz0ovA2Yd4hPESRgP6snjBiFOsKzBJIhBl98RSg3yIRAnqi2PuEs/7uUiNaM1k+rRJkGrq66ejowOPxpByTogD27NlDOJx6Y3sS9jt6nr7+sLJ5UKbgb8HOPCyDDqrJftgaehFRy6C3X6DX+D3Zky8g8AFhDo+QkGwmMyX+XQUOEmWRkek1EgR056eMMCT6cIwYUbRhv584d6VIXICLZVh5lgBHku4jgMrzBCgbIZ9hMuffYaRQJ0Y4lAJgBs4U3ShZxwBkBFYauSuTpQBKkenu6KCtrY26uro0Y0pCcvGPiT4U+sYwCDcilUjMw8ISrMxBHpyAfDzsGNpg2GzACDuNBAVtzEUgOwlRjsQ2HCMep6E7GksQ6Ux4rVqI049KRQb7RbMYah4WbsDLcwR4O6njXhcKj+PjKjwTGoLKBJY0ZCpTBRowx2jgaeaLJGeCpoN1HLRfY4WeIDb8ApSiAEKhEGOFgL7HmWW0N6o2BL8aGbsxjIlK4Ml0Mk4Q5xWCk9KMUkFvf70V+4g+Aj3sJ1GHlc4EFt8QGl0oWTmMzC3FJbgRENiZlNLUTJzf4eNKPAVhCUxFaOgdfC/Hw34igwVhfyE0Yg2LFxH3JOZnmA5mKRKi8fBhtm3blnLMEAUQi8XYvXt3VhcR0D2i5gpfY/DlWxCQOJXyWwgOPtBN4Ymq606HYqSMnFoiuhWQiDgaJ4ixPEtCCpPS6hJcRNF4L2mvaiqBqwvAEkiEQu6apE40JATWY2NNUinvSErAgTAujoixwIqAqsRoHqZpyBAFoGla2v2/jIAbIW09ugeRy3CzAttgpZx5VKEI/eD9cSqUluhwM4tpku9OMu5PHOWh6dWMetw/8RzFiGw22k1lkj8xC3mIWQl6U9KxZMeZSuB8nPSipCQ3tRhK4Fo8VOdJCSTf0wAqPShTpr2cWYNh3st2nCzEwh/wp20k6zbepckeI4AgpL9uyhYg3YEVSHgR6UuzF/ah8qJRyLMBe455eXI/GeVIXI2HvUSIouEymH1PEOdVgkMcOwuxchnuUdNoTG7/x/DRlvDgnYiDxByZjK3Y6JgTShhFFF25jCXMqKI/uytx8zj+FCXQTJwXCXI1nklv6CpACi1YHI02FJZM4jhyfU8LsHABLp4hMCRr0/QFFVrX7IxSxIoQU2reTWjoL9KTBGghzodwUjaGlOHJxGzkQdPX5BrsSuPVXYSFGiMRZzR40ShGJLlxWjZzIBrbpkR0GdEW7xgF1GwieRUeHsOXUsq6l8ggD/5kczwWpaktyYZSvRChAsuMrXBiQpkXsSDrMzLOEd2EnXbitBiJMukYXd4izFHiLDHIE2qM0xfaTSfXoMfQC1ESIcGgKZrJ+Mcb1TD558qRhoxFNfIuxnu/1Yblk6wEFOBlQsgInGkUIE3W86owtjyJTECFaz9mh2JEShKiHIW2HTaRURxG3ztL3ICXz1DER3GlOKxMdBDnL4R4kAF2EiaEVvC16nG0lIo7K0LG5vtwCKBmtaKJafLj/aj05qDhqF4jrucKJHfmDRo5Am8Ym4/JeFYa+uoz2XviyYLpUzJ/8iX85uwmF/eZGPImWK1Wli5dOuwNeRFZgIUP4eQmvGw16KbToROFJ/Dxa/rZRYQgWkaZcfmYoAhaSv1/MVJWPPqSkc6bCB8qx4llfM+m1ZGIGLkzi02Cy21pSC1CaPyJAG8TIk7+nlM663IqQmTim9dkghgaomihpqY67d9TtgDV1dXDnizRjKlG5uO4mYvMLiIcT6DqNqEAR4jRRJx5yGxMqO8vlJ2egO7D6CKV/diRhQKQgTnI7EqYI5XshFdAVzyJ++K4UZ680gg3jRci8CEcaMDLBIeE3SJoPE2AMBpn40zhP5wMtBAnglbQzuSRIKALXQtxQkYWZ6WxmMDkzqeZoarYrSxZnN61mlEUIB009BVrE3ZWYKOeCG8R4kSa8EccjcNGPXudYUHMx4KFwqCt7kdJUV5z0xCAjoYKZCwII7IYj4YqJGwIQxKVThpC4cgRd7wFgXON7MSXCA659ygaLxi78nMmWAm4DMdYopLMZZv0fCCMxp8J8A5hougCVo7EOuxswp6TZ5gNVEDVhreqUhTA6tWrsdvtafMB0sGMN2/BzlKsvEmI94ikZbGJo3GAKMeN5JZNOJhnCE2+zD6zfn7opAjMMTLws/Pip5p7HSgZk4OaYcpipCHhxA4UOlGYl6N4faISMC2BRCUQQ+PPBvvtOcZ2IdfPR0PnBEguMTc7JU92WDJXeJswf0noJxVHV+DN+OlH4ZKUjeLEQQN6iVNWUU5VZWXaY1LGUlpaiixnRyBhOjy8iFyAi1so4sNGODAdwmi8R4QH6OdRfIOFK/kw+iJoQ/LvQdeK2Rrcpo8knYMt0zpJXZmKLEyixg6i0jguuyL9tSwInIeDD+NMKTiKofEiQV6dYJ9Acp76AAp9BdQjMPP70Ld77xFOa9VqwLtG2fZkKoAe4pRWVAy7tU8ZS1lZGRUVFUM+y9SLaR5ThcSFuPgrijgf57De9BAa7xt97vTEicl98CYFeHKGYxHSmBqfOhBSFEcnCr1Z3JeE2X5s6DfaGQ/FaHronniBD+HgPJwpjsGoYQm8YlgIuX42FoNLMRFhNDomqOHrRCPKyK3hgwbHw2QhjkYfCnPnzsFmS88FkaIAKioqUhRANwqBLF4A80WtMMJOf2VYBMNVQvlQ2UGQB+innsikhaKGQ5XBV5eNwGmkb3rZg8KJLCIBJv20K+kbPagT0j47cTtw3giWwFMEGEDN6eplRj0Sr6gCR4kNbpsm+j3IdWRKGOVsubXjRr6vMBo9qCxfviItGQikUQA2m43Zs2cP+Sxd//pMYH6j0rAIPk0RH8FJyTCK4ARxHsHHs/jx5/hlG3mqhsKKMKb4tBWBZUmFOyYFejaz5zGcY4loJ04b8QkRCNMS2D7CduANQvwOHydyPIZKpBTLYw8RnsHPAaL0G9ZTYkhtPEJrftcs3w2mCQGPby5HPlcr8UmxAQT02oqQJLN61aphj0vZ7FutVubPn5/TwSQqgo/gYgU2/kKI/URTCDhDaPyFEK3EOR8XC8bgjMtmkvpQUnZt47lWRZqCnlYjnTcTD7DpB1iGdQjxZ4zMSUbHgsTtgAa8mOQYBNhHlB5UPoGbeVhykqFYhUwl0hBHbMjgbHiTMOVIzDUKuIqRKENCRrdaHIjIZKYMNPQIQxiVMHobsj1EaCKOhE6Xtg3HqOQtI8GBSAVSSlOZRNhHtRFyh07iCE471VVVwx6TkbcvVwM2X6daZK7GQxMx/kwwhW1HAxqJ0ckAH8LJWmxZJeVkgwBqillWNMb8NJMs0pNU0HPCaHKaqRdfRPdDJCo+FT1GvnYCef1MJXCuURfwYlKeAOiWyLMEuBxPynZnrPO1CTvN+FP+HkOjlfhgl2YJPXQooivJMiRqDIIZMDs0pSeu7UFlP1G6URhAIYg25Lm3EacEifXYxqTYzGjYhbiwE+IosSFlwXajs9Umg1lzojcCGtBElJo5dSxZMnx5VVoFkJw2GEajN0NmmkwHJ6FnpZXj4WWCvJEQOjHRj8pT+GkixlV4JqSWWmHow9BplFJ7/WV6X25ElmClI4HYI4pGu5EMlSmKEbEiDOEuaJ6EJBkzRfccHHgQeYFgSi+Cw8T4DQNcjzcn7LZrsNFIjIZRmBoUGOQ77DPo5hM79FjTFFOZiMGIGzEFDOfj2BWshs7IdD0yh4jxMkH6UVmIhc3YqUSaNEKQMDpp6flnbKUqWwvA7XYP+T1qEFMsyXHnVjN0eBEuypB4lVBKtyANvWKtHGnQSZWrCYyhd95JFvTxiJcEVCaRiGqQlrF3OJj5AN4kirB+FIKo2CaBUlpGYDN2ypB4DF9KpmQzcQ4TpcrYMoznWm5EPoGHHUi8Q5gQ6pj2yeNxsNmMVlq5mrsVWJmPxSg5FyaV/1JEt9R6RJEztp6BxWIZ8dgUbNy4MaWneGcG/HljnTAbAttwcANeFqdZ3xRgByFeTqrXHw/MPgbJYRmdH2B8CTeVRjZfIhqJMpBhONDMKahK0s9+VPwTEAkYaRx1WLgEV9qeELm8jhOBC3BxK0VcjYfV2Cgx+jFMpDPY7FT8cdwswZqzd9y8p2JE5DxwYJ4khr2kiHVr1454XFoLYNmyZVRWVtLU1DT4md9o7jEeJ8lIkwV6+u31eHiTMDsIDtHocTReJ8wirCzMgQMK9PBjsmOt3Gj0OdaHpaFvIcqRhjjxBoyOQ94MX2dLGgLJiMEROHcSO/3o9e02zjN6EJrzVY3MQqw5e6nNbWEtMrXIrMaGz6im7Dfi5+Y7aPaeMGPuShYl03YESpCoNDpdzzK4IXK1vU2+p3wgisYhIixeunbE/T8MowCcTicOx1Am21ajGWLNBNJHmebgh3FiReAFAkN2hUFU9hFlfo62Ii3EUxI3ypDGtcc2V+85WIYogLAhvNnMX7KyVYw5mGxI6HUBS7BylBgxYCkWqiZAaMy7sxhEm6YSTHSGRtEIJLRw7zeoxEaDgEkGY8GWwFlZqLX6Y4HZ3boFlc9e8wlKSkpGPD6tAqiqquKcc87hwIEDg58FUGkmPiH9zBNh8qxtw4GCxvMEhjzaI0Tx4Rg3u4rJ259aBGRBYnxFSibbTSJMzsBsUIOc4ozMV4KUYIxnVgLJy2R4spOvIaCv4g7DUoPsx6El/Uw37COMc1Y1F1140ajHprVHrVYrq5KSB1T0biiTQadtmoMbDSdUIlqzzKxLBwFdoSWnnBYZefjjvUMB3ZJInFwNaCCKL4u0YCvCJEaNR0ciyUW+SEQ19O2oD5UBwwoIG+k3ySQcw/1MR6E3EUbjIBG2bNmSthFIMobdTK5cuZKysjK6u7sHPzMbVUx0Y0MwKbJE5mIZ4qiLG6b0eK5vcgC0pigAKWe8bS4jjJeoMBuJ8nv8XIp7sBfiSNAbl0zn1zVzCOiRlLcI0UhskClZQ6MYiQ0G18Rkl9sWEkTgMFF67DY+cfXVw+b/J38nLbZv384ll1wy5LNsGW7GCxlYgS0ltnsyB+mU7WnOsRRrTmg3VHRykMVJacEqsJsIb6bJeUiHzjSK7nR8uc2uzb/Dx18I0UacDuK0E6cDhYNEeQQff8A/JSsJc4U4sIsgS9as4sILL8zoO8MqAFmWWZsUQlDQtwGT1VhDTxWVUiqoTxCnZxw8eTHjHImQEKhOMtvHAxsCZ+BIGz7bR5RABi9qcussMxPudIMAHCWaQmueCAW9HfpDDHBygmomChkieujvmKByxccvp3KY+v903xsWV111FcuWLRvy2RFinCQ+aa+hMw0leS8K+4mO6SHr8X+V7hQKMIGKHDo4VXSO+A/jSrEq4hmErTT0/PR12HAb6cWbcLA4Bz6KqYgYmVk/x4nx2AQULRU6YsBOgsxasphPfvKTGX9vRDlesGABF1xwwZDPImjsMUp2JxpmF9naNK6KJiMcNdbzJqMMKcVzP15IwBnYWZAUtszE+2yGE6/Ew/+iiP9FEZcajL6nowJIFmZBkPAWrcJiLU45tsWoV8g08WqqQwSaiXGAOLfccgsLFy7M6rvDQhAELr30UsrLy4d8vpsIB8a4AmcLAT1XPNn0PU6MjjFoeQE9bpwckjN7GU4E605yRl8RYka+BjNLsgaZ6jxTp40VQpqfnJxXkJi3+HOsXH8X3uI1KX9vJGpQnU1/KMDbBJmzcjk33nBDVt8ddck7//zzueOOO5gzZ87gZ36DR749B3z1maAGmaqkbUA/KruIjCleH0mqBAM9ey+TJp7ZQgC2YmcdNsqQWGhsC7LxVk+1eHVivX0vCl0JPyY9/HihoaFpCiVlW1iy6h8oKlmXcsw7hDlIdFp7TUTgEFH2CQo333wzc+fOzer7o+aUSpLE5z//eUpLS/nSl75EV1cXoIfRnsbPJ/FOaOhFQ0/8WIp1kDvQxH4inIV9TPRdyfAijjsBaLjxlxldeQIG2aVzCq7kmUBvkqor2D4UdhNhD5EhyVYlSJyBg9XYshLMlGiIGiMwcIjyyu24PYtYsvKb7Nv9D/gHTiWvRdCoJ8JSrNNSCej5LBov42f1WVu5+aabsj5HxvNyzTXX8LWvfW0IYWgjUT5IKMecSCzBmpJH34XCvjQNS8eCic5utCBQgjithb/LaAbzS/r4L/p5iSBdBsmn+XOUGL/Dx1tZ8vA4EVIstHjcB2homorLU8ecBZ9Gkp1DjmkkqhNj5HuCJgi7CNHpsvM3f/031NbWZv39rBTjrbfeOiQ3QAFeJUjLBE+w2eByTVKttoZu5mWTXQf5S6edaqZ8NtArNoPsJEwz8RFrFiJoPGu0IsvE4jKtKHvSk+vv/YBYtA8Q0DSN0oqzcHuGdrbqN4hAphsEoB2F1wnw8Suv5KorrxzTebJSAGVlZdx5550sXrx48LMuFN4iPAGctakD3YQ9xVPfTpzDWSYnBbOoHpvOMJ1yWsLvYz1PDI32LGJDEaPOI1NnsssIhSYiFDhOONxmNLPRkGUPJWWbhxxjcjFMt+etk7X68S5cyFf/7u8yyvpLh6y3RqtXr+YHP/gBxcXFg5+9R5jGCeY7N/njkq0ABXiNUFbU253EU6iuptoecbzedbMeYgdBHmaAlwniH2PYzKzd8KSZRUGQ8HiX4y1eiyAkCTAaf8Q/al6JZpx7Xko4VQPNTAoGQRAprzofi3XkCripDgFd5g7ZBP72K19h3bp1Yz7XmN77yy67jDvvvBOPxwPo2vxFghMeFZDQU4OTs+tajLz+TK5tMsEmQu/fNnk19uOB6WEPodFvFMP0G4Uxp0RhdGjAG4R5hgAfEOFZAuzI0CRPB0uabsqS5GD+os+zauOPWbn+X6id90kEYeg8d6LwGqFRs0tFSKFKB13BCIJe2BuP+fAN7ENTx5ohUvgQ0PkLX8LPRZdeyqc+9alxnW9Mb73FYuHGG2/kt7/9LTt27AD0uPyLBLkadwrNc66gotdzL8bK7iFccKTsD4dD3ODnS57UTL+fT5ir9n6ivE14kDLb/NsKbJyHc1RHo94QRUvpNnSEKFGcOWpDCqJopaR8KzZ7DaAyf9HnCIda6Gp/achxu4mwDCvrsiTkVJQQbc1P4utvYKC/nlDgGL7+BhRlaFu7wn+ymcHk+n+JAMV1dfzDN7+J1+sd1znHvOwVFRXx1a9+lT179tDT0wNAAxEWGASIEzXpVgQ+gos4GieIY0FgE3ZmI2f08kgG0UQizFTbQnbQaej8BTsIcoxY2t32XwhShcQm7KOez5eGRCPXTsp43EfAfwRv8Ro0TUW2eKhb9hWikS4G+vacOs4I163AOuzioaCHvIaMV41x8thDI47BgsACLIg5vrd8QAVeJsAxr53/uOMO1q9fP+5zjmvre8kll3DLLbcM/h5B4xn8vE94wvbUZoHQtXj5LMX8L4rYnqat1Ug3fBYONhosrfOxcLFBSlqoL4jeVy7Mb/BxeBjhB/0FyYQZxzw2WWF6DZ79sUAg1QegaSqRUBuapg7+7nDOpnbe9UjS0HDdAaLsG8YhKKKnftePIeS8GhursRXss80GuwjzJhE+/6UvcfXVV+fknOOSU0mS+MIXvsDy5csHPwsZ/eSOTWDZsFkjUGU0isg2oaQMiatw8wWK+SuKWJszozf3MOPrfyY4hGc+HZwGf0Km501G+Tjp0CqQUkq3Q8EmNO2UUtI0jfKq7ZRWnD3kuDAabxMelnDG5ATMFHYE1mPn4jTFWFMNAroC/BM+PnrNVXz17/5uRKbfbDDuhXrx4sV861vforS0dPCzLhSeJTBmr3ImGA+lk+61FnAZOfmFvjocJJrCzQ/61mUVNjYYveevxM2iDIg6TV9CLId3rqH3MnCM+kppSJKTWXOvxmorG/KXXhSiw6QKVxgMwekgoVNxu43OPGuw8Um8XDUi8crUyMrQy3zj/I4BFm/ayJ3fuyOlNmc8yInr+7rrrqOrq4s777yT9vZ2AJqIc4QYawvY/CrUcSUilsZZB7AQCxfhohZ5MEMum31uL0qK531s/ZBOwVSo/lGO0zSFopL1FJVsoLPt+VHPq6I33LgYF8/gH5JDaHbjKUfCYfhyzC7Nw9N/aSiSTnorKSEK1U1o9vd7mgHK16ziP3/xnyxdunTc501ETrbqoijyN3/zN9x3332DpYimY2f6BmQmB2ZPu0S4EbkEFwuxIBusgWNpN5V4vIjOYiQiDDbizI0fJ71wCYKIKFozPouEXhVakbRmKeiNWJZhYzYyRYhYDOEf7q47yzfz7sbv8u7Gf6SzYssYZm7iIaA7an/PAAPlJXz3299mw4YNOb9OTn11F198Md///vcHtwOHjZJdk355BrlBESIVGUY9kmEqi7403+5F5QhRjhDjCDGajISp8Tw7TYuT6m4UUJQwkXB75udB9/ssTPJxmMVHagIx6EhnUSQHRxZeQ2f5JjrLN9BYdwMRa8mo35xMJAp/e2UJP/nJT7jqqqsm5Fo5z365+uqr8fv9fP3rX6e7u5vf42cjdtZgm5DefjPIDGalng+VVuLsTfKoq8BzBIasCBICK7FyyTiISIKB48RjPizWUkwhEwSJvu638Q3sSxnjSBAhJdkok+8NuU9RJmZxI2gKgqbhd88l6KrFHu2ZxJ5Lw8MU/icYoK2imLt//GNuvPHGCbtezhWAJEnccsstVFVV8Y1vfIM9e/ZwnBh+VM7DOf4LzIAQGgOoODPkxTebRbxOiANE6UdNW7uR6hTUPfNlRl/GkWCu0BVIQ+jWVCU8JAoA0Nv1Bscaf4ESDwz5vNwo+NFGuEbqZ1oWVPUCcjyI29/EgHexcUbBEPz8L03mnv8PDNBeUcyPf/SjCRV+mKAUeFEU+djHPsavf/1rvvnNb1JUWko9YSIFoWOnPnpR+B0+3iGcEcFGBI0/EeB1QnSjZF24tZtIRvRaMkKKp15RwsTjAb1gRxDp7tjB/j3/iH9g/5DjJGAltlHDkOn6RrZnfEcaEVsZEXs5hSDwyfeVaPbffffd3HzzzRN+3QlNgF+3bh1r1qyhpqaGv//a1/lLKMB23MgU2vQXLiwIzEbmYEJJqwYcI8YJYiwjyqW4hiVFMUN+zeMgx+pGoQMlo14GyStKNNLJQO/7OF3z6Wp/icaGfyES6Uz53kpsrMigNbfZeDUxgpGZL0RDFW0cXPJpekpWIxhFRIIWQ1Kj5NNLZa78Txgr/90/+Qk3ZEntNVZMeAWMKIrcfPPNHDhwgP/+xS8gGuBsnDP+gAwhAOuw8z6RlCw/k6Z9AJVLcFE3DGOwHZFSpBGzBFdiY57xOhwhNqSGPoJGC3EWjZJkJKLzNiRC01ROHv8f/L5DdLY+RzTak/K9WchcgAvXiN57s+Ou3m03W2p6AY3Ois20V5556nyCSFnPbtz+przZpiZH5R8YoLOyRN/zT5LwwyQoAACv18sPfvADXG43P/3JT2gLD/BxPBQhTrs67VxDQ1/1LsTFU/jTZsOdIMbv8XMz3pQut6dab+tFPu1GT4XEs7gQOQcHdYYB7jB8BYnniWSwU9aAeci4EYeMM+BrJOBrTPudpVi5EBeVSBP2Lgio+NwLaKy73nAAqmiCSHHffhY1PoSshPKiAER0S+6PDBCqLOfue+7hk9dfP+ljmBQ4nU6+8+1vc8+//RvtVSU8Rj+tk0QqOh2wDhs34WU99rT75HbivEkoraGvAfOxcANersaDPemxVyFRg4xqhNMq06QERwwm5dEUwCwsrM3AlAdYi41r8DA7i34MZjl0IqQRj1eJWIrZt+yz9BctGTT95XiQuiMP4/U15kX4NfQGMb+lD8/a1fzXL3856cIPk8yDYbfbufXWW/npvfcSWzSfh+jjkLHSzCiC0bEQC1fj5mO4U0xtgPeJ0DoMuYbOS6jnyCfPdTXykJRokdTnsYsIv6SflwmOmEIsA2fjSInXDz1Gr+C8DHdWVZhmuvGyBPVUgpTSgs2EgErIXsWBpZ+hu2y9IfygCVDT+grlXe+iTTIVjIDus9hJiEdEPyu3b+fX993HZZddNqnjMJEXFoxrr72Wuro6brv9dv7ntdf5CC624JhxDo4CFV14zsDOPGQeYmBI49QI2ohcfJB+fi2GUjilAFKVRACVgNFRaRYyy7GmvZKGHqu/Eg/PEOAQ0SEKw4HAeTg5A8eY6jAsCFyMi3lY8KOyEAtz0vg+BFQG3AupX/nX9JasPDU+QaS8axeLDj+IrIQndfUX0Cn1XyTAu0KU6z/9af75zjupqamZtDEkI280OBs3buQ3Dz/Mt779bX57/wO0x2N8OMOuuac7zFVkImAKcDkyJ9IkckfR6B2l5FgzthHX4eEosUHSVrMScz6WMTdh0dBTobcYnAfJ2X+C8Uln+Wb2L72VAW9dwsovYQ93sOTQr3GGWtFG3DzkHieJ80d89JZ6+caX/56/+eu/pqQkv/RleeXBmj17NvfcfTcrVqzgrn/9V1o7e7gED/OxFEhqRmEgkfVHAQ4R4xn8Q1Z/0J19Y2lvnrgGmk7DrdjpIJ7ibbciUDKs4GhogkTM4kaOh7CrUVakMc+HL9LJDKkpvxoCGhoCQUcVbdXn0DTnYwRctQnCL+IdOMzCI7+huG//pAm/SZj6ARGex8/s1av40fe+x6WXXookTa4CSoe8E+F5vV7+7m//lrVr1vC1r3+dBz/YzTYcbM2A2mo6whTGOAbpJTqHYQiVNhQOE6WBaNpowFrsVI6hwakfFYVTDiENWI8dGwLvEabdaFMuG58vxJLWAlEkO4cXXkdH5Zm4/cdZevBXOENtE2Rm60KvCjJRawlhexmt1efQWbEZv3sumiAOCj8IlPbsZdmB/6Kkr2HS9v1miO8VArwrK1x02RX84Pvfz3lF33iQdwUAeq7AhRdeSF1dHd/+znd4/NHHOBbp42I81BhDPB0UgYDO6FNvtMI29849KITQCIzghV+Kle04xmRaHyZGNwqVCSFECd1LvxzrIImqTsyZvmhYQGHAu4hj868kLrsY8C6kaOAQC48+Qi5dvGamQEx2MeBdTMus8+gpWUnUWkLUqvPjCZqKoGlogoioxqno3MmKff9hKKOJF37TUjtIhBfwE64o4//767/m9ttvp6ioaMKvnw0KQgGYqKur4+f/8R+ce+65/PCHP+TXhxo5FxfrsWMfJUlkqsNk/nkUH0ezKKIWgOVYuSiDgh2TXrsEcQi7UAzNWOOHwnQ6FiUI8Ejnj1lcqKIFQVPQBJGwrSxnq7+AiiJaidjK6C1ZSfOsD9NftISoVRcoQdMSzH39mq5AM/OPP0FN6yvYor2TIvwielXlqwR4T4iyafuH+OY3/p4Pf/jDBWHyJ6OgFACAx+Phc5/9LGedeSbfu+MOnnri9xyK9HMuLuZMc9/AbiJZCb8EbMTOJbgz2i6ZLce34KALP2FDJJZjHZETMeMYvaoaSTb6i95ftISYxYMl5iN7K0AbvCNFtNNdto6TtRfQX7SEiK0ERbIlCb3OYCCqMRyhDmpaX2V283O4AicGzzaRENEdpPuJ8hJ+IuWl3P75z3P7bbdRWVk5odceDwRN0wpWnvx+Pw8++CA//bd/48TeBjZhYwtOSqZhpEAAniXASwRHPdaKQDkSW7APWkfZzIcCHCVKGwoeRBZjxTVOf4uARtBRzc7N/0zANRtB05CUMCsafsac5mdHXH31K5/ar6uCTFx24XPPw+eto7d4GZ0VW4hZPIBu3g9+0zDzi/r34wh1UNK7l/Ku93AFmxFQJ3zVNxekk8R5jQCHrAJbz/0Q/+drX+P8889HFAu75UxBKwAThw8f5t9/9jMeuP9+LJ1dnI+HpVixTqNtgYCeFvpbfIPltKVIuA1lZ0HvjFSEyCxkqpHxGvvxsTzAxNcyd+x4AntX3UbTnI8NbgO8A4dZu/tf8foOp/2GhkzYXk7YVoYqWen31jHgXUTQWUPANYeYxYUmSIbQG+W7gk5rYo32U9y3j4qud6luexVrtE+v80ebFMHX4/oa7xLiVQIUzZ3LV7/6VW6+6aa8h/cyvo+poABAZ5N9+eWX+dGPf8wrzz3PnKjCNlwswIrMxMXFJxstRq9DC7DI6IhsZkpak+i/Cu3BCah0lW3gvXV/T8ziHRyh29/ErJYX8fiPnxq1BhFbCX3Fy+gtWUnEXoaGiCLZjC2EliD0+tk1QcASC+AKnMQ70Mislhcp7j+IpJoJPZOT1GN2ZtpLhDcJMOBxceUnPsHtt902rjZd+cCUUQAm+vv7+c1vf8v//e//puGtnazRLGzCSQ3ytGj+kPgaF6KQjwZVlDmw9FaOzr9qyP5c36/HEExtpoEqSmiCnv85VNj1mdAEAD2cZ4kNUNb9AbObn6Oo/wDWqA8BZVJTeUUgBhwmypsEOCILnHfRhXzpC1/g/PPPx+Fw5Hv6s8aUUwAmmpub+a9f/pL/9//uo+voMVZjZwsOqqeJIpiqEFDxu+bxwZqv0le8FL3mfjT77JRZrysDFUkJY4n5KO4/QElvPcV9+/H4jiErwUld7eGUxdVGnDcIskeIs3LzRj732c9y1VVXDaHEn2qYsgrARH19PT+86y6eeuopIl1drMXORhzTxiKYmtAIOao5Nv8KOiq2EHJUo4rpKz1MZ6Ej1I534DCWeBC37yiuQDPOUBu2SDeSGuEUddfkCb6I7jBtJc67BNlDjPK6BXz605/iC5//QkF79zPFlFcAANFolDfffJNf3XcfTz75JJGuLtYYimAWMhLTx0cwVaBn6UmEHNV0la0nZnGTqgAERDWGx38cl78Je6RbT+IZTHEW81anH0HjMDHqCXGQGOWL6vjkJz/JDZ/8JMuWLdMpzqYBpoUCMBGNRnnnnXf4z//6T5588ili3b2sxMJWnFQiDyY9TJsbngIQMvZkCHkR9lNX1xFB4xgxXifAcVGjav48brrxRj51880sXrw4b+ObsPueTgrARCwW47333uPJp57k4Yd/Q8ehw8xDZKNRRuo0Hve0u/EZZA1zm+hDpYEI7xOkwyKzavMmbr7pJi74yEempeCbmJYKIBEHDhzg0cce5amnnmbv2+9QEo+zASdLsVFqNBad2R6cXjA9CTF0JqV6wjQQIuTxsGbjRm6+8UYuu+wyqqqq8j3UiZ+L6a4ATHR3d/P000/z2O9+x6svv4zWP8AKrKw2/AT2GatgWsMUehWd3KSFOO8QpJE4JbNrueiSS7j2E5/gjDPOwOv15nu4kzcvp4sCMBEKhXjttdd48qmneObZZ2lrPEKZorAcO8sNq8Caxc51BoUN08QPo9FGnD2EaCRCxOli0eqVXHH5FXz0kktYs2ZNwaftTgROOwVgQlVVjh8/znPPP8fTf3yGd3fuJNjaTg0iS7GxEBtlSFiYmgk5pysSA4VhNDpROEqEvYTpQKO6biHnbt/OZZdeytnbtlFRUZHvIecVp60CSEQkEqGxsZHHn3icPz7zLPsbGoj39jEHiWXYWYCVEsRBKsoZhVBYSEyPDqHRSZxGIhwgQqcoUFRdxboNG7jwggu49GMfY8GCBaflap8OMwogCb29vezZs4eXXn6ZP7/4Ig279xDr7WU2IkuwMxur0YteKNic/NMBiXt6HyqdxDlKlENE6BIEimZVs3HzZi44/3zOOecclixZMiVTdScaMwpgBPT19VHf0MBLL73Es3/6E4f27SPY3UupprEYG/OwUo2MCxGZwi3SmQ5IXOWjRnPUY0Q5QpSTxAhYLBRXVbJh00YuufgSzt62jbq6uhmhHwUzCiBD+Hw+jh49yquvvspzL7zAvoYGWpuasIYjVCIyDyuzsVJhlPBaZiyEMSO5ICqGhh+Vk8RpIsoJonSjIXg9zFu4kPXr13PxRReyedNmZs+ejc2WWWOSGcwogDEhHA7T2trKu+++y8533mbXrvc4sG8ffe0dWGNRKpGYg5VqLJQjUWJEFhJJN2cmXYeQ9K8ChIz+A90otBClnThdqMRdTipqa1m9ehUbN2xk65YtrF69mvLy8oKk25oKmFEAOYDf76elpYX6hgbeeOMNdr69k4MHDuLr7UUIhSlFoAyJWqzUIFNsWAkyAhJDV7vEf6cTkgVdQ2c+DqHiR6UXhTZinCRGDyoDgoDN66FyVg1Llixh69atbN2yhcWLF1NVVTWzyucIMwpgAhAIBDje1ETT8ePs2bOHvQ0N7Nmzh5bmZgI9vVhjcZyaSjkyFUhUY8VmKAkZASciIqk97wrZckhXp6ehr+gKGkE0omj0oNCHQhcxOonTg0pQlLB7PTiKPMyfv4C1a9eyfOlS1q5bx6K6OkpKSrBardkPagajYkYBTAIURaG7u5vW1lYaDx/m5MmT7K2vp6Ghnu6ubjqaW1BjMYRwCAtQjoQdkVlYkBGoRMZhOBqLkBDQlYMlg+KZ8fH8jQwV3SFnNr/oN2jLe1HoRyGMSisxwqh0Gfx8qtOO1eGguraWpUuXMnv2bFavWsWiujpqa2spLy+nqKho2lTbFTpmFECeoGkaPp8Pv9/PyZMn6e3rY9d7u+jt6eWD3bvx+/00HTtGPBYjNOAjHokiaxoug1zDjUgNckLLLZmSNCTPJUjYxlBlF0GjL4ksXCfFiBJARUDAj0Kr0Y9YAQKizixsdTuxOp3IssycefOoqqpi9apVlJWXsWHdetxuN3PmzMHr9c6s7HnGjAIoQESjUcLhMG1tbcRiMQ41NtLf38/AwAB79u5FURQikQj79+8nHA4jCAL9PT0M9PaSvG5LwTCCFs96DKLFRsw6tOmmIAiUVVfhcDrRNI3S0lIWLaoDBDweD2tWrUK2WJhdW8us2llIokR1dTUOhwOLxZL1GGYw8ZhRAFMM5uNSFIW+vj4URUEQBNrb2+nq6ko5dt/+/XT39GRnUmsa8+fPZ87s2SS+HoIgMGfOHDweD5qmYbPZ8Hq9CIIwY7JPUcwogBnM4DTGTEL0DGZwGuP/B8DI1hfJLnhyAAAAAElFTkSuQmCC";
const __logoImg = new Image(); __logoImg.src = LOGO_EARL_DATA_URL;
function logoBanerMaterial(tloHex){
    let c = document.createElement("canvas"); c.width=256; c.height=160;
    let g = c.getContext("2d"); g.imageSmoothingEnabled = false;
    g.fillStyle = tloHex||"#fff1e8"; g.fillRect(0,0,c.width,c.height);
    g.fillStyle = "#1c2024"; g.fillRect(0,0,c.width,4); g.fillRect(0,c.height-4,c.width,4);
    g.fillRect(0,0,4,c.height); g.fillRect(c.width-4,0,4,c.height);
    let tex = new THREE.CanvasTexture(c);
    tex.magFilter = THREE.NearestFilter; tex.minFilter = THREE.NearestFilter;
    let drawLogo = ()=>{
        let iw=__logoImg.width, ih=__logoImg.height; if(!iw) return;
        let pad=12, maxW=c.width-pad*2, maxH=c.height-pad*2;
        let s = Math.min(maxW/iw, maxH/ih);
        let w=iw*s, h=ih*s, x=(c.width-w)/2, y=(c.height-h)/2;
        g.drawImage(__logoImg, x,y,w,h); tex.needsUpdate = true;
    };
    if(__logoImg.complete && __logoImg.width) drawLogo();
    else __logoImg.addEventListener("load", drawLogo, {once:false});
    return new THREE.MeshStandardMaterial({ map: tex, roughness: 1.0 });
}

function upgradeTasma() {
    let nastepnyLvl = ulepszenia.tasmaLvl + 1;
    let koszt = kosztyTasma[nastepnyLvl];
    if (zarobek < koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    zarobek -= koszt; ulepszenia.tasmaLvl = nastepnyLvl;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    playSound(520, 0.15, 'sine');
    let btn = document.getElementById("btn-up-tasma");
    document.getElementById("txt-up-tasma").innerText = `🚀 1. Taśma (Lvl ${nastepnyLvl})`;
    if (nastepnyLvl === 4) { btn.innerText = "MAX"; btn.disabled = true; btn.style.background = "#5f574f"; }
    else { btn.innerText = `Lvl ${nastepnyLvl + 1}: ${kosztyTasma[nastepnyLvl + 1]}zł`; }
}

function upgradeLaser() {
    let nastepnyLvl = ulepszenia.laserLvl + 1;
    let koszt = kosztyLaser[nastepnyLvl];
    if (zarobek < koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    zarobek -= koszt; ulepszenia.laserLvl = nastepnyLvl;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    playSound(520, 0.15, 'sine');
    let btn = document.getElementById("btn-up-laser");
    document.getElementById("txt-up-laser").innerText = `🎯 2. Laser (Lvl ${nastepnyLvl})`;
    if (nastepnyLvl === 4) { btn.innerText = "MAX"; btn.disabled = true; btn.style.background = "#5f574f"; }
    else { btn.innerText = `Lvl ${nastepnyLvl + 1}: ${kosztyLaser[nastepnyLvl + 1]}zł`; }
}

function kupUlepszenieJednorazowe(id, koszt, kluczUlepszenia) {
    if (zarobek < koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    zarobek -= koszt; 
    
    if(kluczUlepszenia === 'kawa') {
        energiaPracownika = 100;
        document.getElementById("energy-bar-fill").style.width = "100%";
        document.getElementById("dymek").innerHTML = "☕ Wypito pyszne espresso! Energia na maksa!";
        playSound(880, 0.1, 'sine');
        document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
        return;
    }
    
    ulepszenia[kluczUlepszenia] = true;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    playSound(520, 0.15, 'sine');
    let btn = document.getElementById("btn-up-" + id);
    if(btn) { btn.innerText = "ZAKUPIONE"; btn.disabled = true; btn.style.background = "#5f574f"; }
    if (kluczUlepszenia === 'autoTasma') { document.getElementById("controls-hint").innerHTML = "⌨️ Auto-Taśma aktywna! Wpisz resztę i kliknij [ENTER]"; }
}

function wzmijKredyt() {
    zarobek += 1000;
    aktywneKredyty++;
    playSound(880, 0.2, 'sine');
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    document.getElementById("dymek").innerHTML = "🚨 Przyznano pożyczkę korporacyjną 1000 zł!";
    document.getElementById("dymek").style.borderColor = "#ffef00";
}

function podniesPrestiz(koszt) {
    if(zarobek < koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    if(prestizPoziom >= 5) return;
    zarobek -= koszt;
    prestizPoziom++;
    mnoznikPrestizu = 1.0 + (prestizPoziom * 0.1);
    document.getElementById("val-prestiz").innerText = "Poziom " + prestizPoziom;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    playSound(900, 0.25, 'sine');
    
    let offset = prestizPoziom * 0.3;
    retroBox(-2.0 + offset, 1.1, -0.7, 0.2, 0.2, 0.2, generujPixelTexture("standard", "#ab5236"));
    retroBox(-2.0 + offset, 1.25, -0.7, 0.25, 0.25, 0.25, generujPixelTexture("liscie", "#008751"));
    
    if(prestizPoziom === 5) {
        let btn = document.getElementById("btn-prestiz-up");
        btn.innerText = "MAX POZIOM"; btn.disabled = true;
    }
}

// INICJALIZACJA SYSTEMU TASKÓW
function aktywujZadanieZaplecza() {
    aktywneZadanieZaplecza = true;
    tasmaPracuje = false;
    let losoweZadanie = pulaZadanKorpo[Math.floor(Math.random() * pulaZadanKorpo.length)];
    nazwaAktualnegoTaska = losoweZadanie.nazwa;
    wymaganeKliknieciaTaska = losoweZadanie.klikniecia;
    kliknieciaTaska = 0;

    document.getElementById("task-tab-counter").innerText = "1";
    document.getElementById("task-alert-box").style.display = "block";
    
    renderujTaskUI();
    ustawDymekStatusu();
    pokazKase();
    playSound(200, 0.4, 'sawtooth');
}

function renderujTaskUI() {
    let obszar = document.getElementById("task-display-area");
    if(aktywneZadanieZaplecza) {
        let procent = Math.round((kliknieciaTaska / wymaganeKliknieciaTaska) * 100);
        obszar.innerHTML = `
            <div style="font-weight:bold; color:#ff77a8; margin-bottom:10px;">${nazwaAktualnegoTaska}</div>
            <div style="font-size:12px; margin-bottom:12px; color:#fff;">Klikaj przycisk poniżej, aby ukończyć zadanie!</div>
            <button class="action-btn" style="font-size:14px; padding:10px 20px; margin:0 auto 10px;" onclick="kliknijTask()">WYKONAJ PRACĘ (${kliknieciaTaska}/${wymaganeKliknieciaTaska})</button>
            <div class="progress-bar-container" style="border: 2px solid #ff77a8;">
                <div style="width: ${procent}%; height:100%; background:#ff77a8;"></div>
            </div>
        `;
    } else {
        obszar.innerHTML = `<span style="color:#ab9b8e;">Brak aktywnych zadań. Obsługuj klientów dalej! (${licznikDoTaska}/10)</span>`;
    }
}

function kliknijTask() {
    if(!aktywneZadanieZaplecza) return;
    kliknieciaTaska++;
    playSound(600, 0.05, 'triangle');
    renderujTaskUI();

    if(kliknieciaTaska >= wymaganeKliknieciaTaska) {
        aktywneZadanieZaplecza = false;
        licznikDoTaska = 0; // reset licznika do taska
        document.getElementById("task-tab-counter").innerText = "0";
        document.getElementById("task-alert-box").style.display = "none";
        
        let premia = 150;
        zarobek += premia;
        document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
        
        renderujTaskUI();
        ustawDymekStatusu();
        pokazKase();
        playSound(880, 0.25, 'sine');
        document.getElementById("dymek").innerHTML = `📋 Ukończyłeś obowiązki! Manager przyznał premię: +${premia} zł. Wróć do kasy!`;
    }
}

////////////////////////////////////
// THREE.JS INICJALIZACJA SCENY
const scene = new THREE.Scene();
scene.background = new THREE.Color("#d9cb93");

const camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(0,2.1,2);
camera.lookAt(0,1.6,-4);

const renderer = new THREE.WebGLRenderer({ antialias: false, powerPreference: "high-performance" });
// Renderuj w niższej rozdzielczości i skaluj — efekt retro pixel-art
const PIXEL_SCALE = 0.9;
renderer.setPixelRatio(PIXEL_SCALE);
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.domElement.style.width = window.innerWidth + "px";
renderer.domElement.style.height = window.innerHeight + "px";
document.body.appendChild(renderer.domElement);

// Bogatsze, ale wciąż retro oświetlenie
const ambientLight = new THREE.AmbientLight(0xfff2cc, 0.65); scene.add(ambientLight);
const keyLight = new THREE.DirectionalLight(0xfff0c2, 0.95);
keyLight.position.set(6, 10, 4); scene.add(keyLight);
const fillLight = new THREE.DirectionalLight(0x88aaff, 0.35);
fillLight.position.set(-6, 6, -4); scene.add(fillLight);
const rimLight = new THREE.PointLight(0xff77a8, 0.6, 18);
rimLight.position.set(0, 3.2, 2); scene.add(rimLight);

function retroBox(x,y,z,w,h,d, material) {
    let b = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), material); b.position.set(x,y,z); scene.add(b);
    let edg = new THREE.EdgesGeometry(new THREE.BoxGeometry(w,h,d));
    let line = new THREE.LineSegments(edg, new THREE.LineBasicMaterial({ color: 0x1c2024 }));
    line.position.set(x,y,z); scene.add(line);
    return b;
}

let matPodloga = generujPixelTexture("podloga", PALETA.podloga);
matPodloga.map.repeat.set(25, 25); matPodloga.map.wrapS = THREE.RepeatWrapping; matPodloga.map.wrapT = THREE.RepeatWrapping;
retroBox(0, -0.5, 0, 40, 1, 40, matPodloga);

let matSciana = generujPixelTexture("standard", PALETA.tlo);
let sciana = retroBox(0, 2, -8, 20, 5, 0.2, matSciana);
retroBox(-6.0, 2.0, -3.0, 0.2, 5, 12, matSciana); 
retroBox(6.0, 2.0, -3.0, 0.2, 5, 12, matSciana); 

window.matRegal = generujPixelTexture("standard", PALETA.regal);
for(let z = -3.0; z >= -7.0; z -= 1.8) {
    retroBox(-4.2, 1.2, z, 1.0, 2.8, 1.2, matRegal); retroBox(4.2, 1.2, z, 1.0, 2.8, 1.2, matRegal);
}

// === DEKORACJE SKLEPU ===
// Produkty na regałach (kolorowe pudełka)
let kolory = ["#ff004d","#ffec27","#00e436","#29adff","#ff77a8","#ffa300","#83769c","#fff1e8"];
for(let z = -3.0; z >= -7.0; z -= 1.8) {
    for(let yp = 0; yp < 3; yp++) {
        let y = 0.5 + yp * 0.85;
        // lewa półka
        for(let zo = -0.4; zo <= 0.4; zo += 0.4) {
            let c = kolory[Math.floor(Math.random()*kolory.length)];
            retroBox(-3.7, y, z+zo, 0.25, 0.35, 0.25, generujPixelTexture("standard", c));
        }
        // prawa półka
        for(let zo = -0.4; zo <= 0.4; zo += 0.4) {
            let c = kolory[Math.floor(Math.random()*kolory.length)];
            retroBox(3.7, y, z+zo, 0.25, 0.35, 0.25, generujPixelTexture("standard", c));
        }
    }
}

// Lodówka napojów po prawej (przed regałami)
retroBox(5.4, 1.3, 0.5, 1.0, 2.6, 1.0, generujPixelTexture("standard", "#1a1a1a"));
retroBox(5.0, 1.3, 0.5, 0.05, 2.4, 0.95, generujPixelTexture("standard", "#29adff"));
for(let yp=0; yp<4; yp++) {
    for(let zo=-0.3; zo<=0.3; zo+=0.3) {
        retroBox(4.95, 0.5+yp*0.55, 0.5+zo, 0.1, 0.3, 0.2, generujPixelTexture("standard", kolory[Math.floor(Math.random()*kolory.length)]));
    }
}

// Stojak z gazetami przy wejściu
retroBox(-5.4, 0.6, -1.0, 0.8, 1.2, 0.5, generujPixelTexture("standard", "#3a2210"));
retroBox(-5.4, 1.0, -0.75, 0.7, 0.05, 0.3, generujPixelTexture("standard", "#fff1e8"));
retroBox(-5.4, 1.05, -0.75, 0.6, 0.02, 0.25, generujPixelTexture("standard", "#ff004d"));

// Kosz na śmieci
retroBox(-5.5, 0.4, 2.5, 0.5, 0.8, 0.5, generujPixelTexture("standard", "#444"));
retroBox(-5.5, 0.81, 2.5, 0.55, 0.05, 0.55, generujPixelTexture("standard", "#222"));

// Roślina doniczkowa
retroBox(5.5, 0.35, 2.8, 0.45, 0.7, 0.45, generujPixelTexture("standard", "#6e4020"));
retroBox(5.5, 0.85, 2.8, 0.6, 0.4, 0.6, generujPixelTexture("standard", "#00e436"));
retroBox(5.5, 1.15, 2.8, 0.4, 0.3, 0.4, generujPixelTexture("standard", "#00b020"));

// === BANERY Z LOGIEM EARL! SHOP ===
// Ściana tylna (za sklepem) - 3 duże banery
retroBox(-3.5, 3.0, -7.88, 2.4, 1.5, 0.04, logoBanerMaterial("#fff1e8"));
retroBox( 0.0, 3.2, -7.88, 2.8, 1.8, 0.04, logoBanerMaterial("#ffec27"));
retroBox( 3.5, 3.0, -7.88, 2.4, 1.5, 0.04, logoBanerMaterial("#fff1e8"));
// Boczne ściany - banery z logiem
retroBox(-5.88, 2.8, -2.0, 0.04, 1.5, 1.0, logoBanerMaterial("#fff1e8"));
retroBox(-5.88, 2.8, -5.0, 0.04, 1.5, 1.0, logoBanerMaterial("#ffec27"));
retroBox( 5.88, 2.8, -2.0, 0.04, 1.5, 1.0, logoBanerMaterial("#fff1e8"));
retroBox( 5.88, 2.8, -5.0, 0.04, 1.5, 1.0, logoBanerMaterial("#ffec27"));

// Lampy sufitowe (białe panele)
for(let z = -6; z <= 4; z += 3) {
    retroBox(-2.5, 4.4, z, 1.2, 0.1, 0.4, generujPixelTexture("standard", "#fff1e8"));
    retroBox( 2.5, 4.4, z, 1.2, 0.1, 0.4, generujPixelTexture("standard", "#fff1e8"));
}

// Stojak na słodycze przed ladą (cukierki przy kasie)
retroBox(-2.5, 0.7, 0.4, 0.6, 1.4, 0.4, generujPixelTexture("standard", "#1a1a1a"));
for(let yp=0; yp<3; yp++) {
    retroBox(-2.5, 0.5+yp*0.45, 0.4, 0.55, 0.1, 0.35, generujPixelTexture("standard", kolory[yp%kolory.length]));
}

// Znak EXIT nad wejściem (po lewo-przodzie)
retroBox(-5.8, 3.5, 3.5, 0.05, 0.4, 0.8, generujPixelTexture("standard", "#00e436"));

// Wycieraczka przed wejściem
retroBox(-5.0, 0.01, 3.0, 1.5, 0.02, 1.0, generujPixelTexture("standard", "#5c2018"));


let lada = retroBox(0, 0.5, -0.4, 4.6, 1.0, 1.2, generujPixelTexture("standard", PALETA.lada)); 
retroBox(0, 1.01, -0.4, 4.6, 0.02, 1.2, generujPixelTexture("standard", PALETA.blat)); 
retroBox(-0.7, 1.03, -0.3, 2.4, 0.01, 0.6, generujPixelTexture("standard", "#12131c"));

retroBox(0.8, 1.03, -0.3, 0.4, 0.01, 0.4, generujPixelTexture("standard", "#4e5768"));
let laserGeom = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(0.8, 1.05, -0.48), new THREE.Vector3(0.8, 1.05, -0.12)]);
let laser = new THREE.Line(laserGeom, new THREE.LineBasicMaterial({ color: 0xff004d })); scene.add(laser);

retroBox(1.5, 1.12, -0.2, 0.6, 0.15, 0.5, generujPixelTexture("standard", "#2c303e"));
let klaw = retroBox(1.5, 1.15, -0.05, 0.5, 0.05, 0.22, generujPixelTexture("standard", "#181a24")); klaw.rotation.x = 0.3;
retroBox(1.5, 1.35, -0.3, 0.55, 0.30, 0.15, generujPixelTexture("standard", "#2c303e"));

let canvasPOS = document.createElement("canvas"); canvasPOS.width = 256; canvasPOS.height = 128; 
let ctxPOS = canvasPOS.getContext("2d"); let texPOS = new THREE.CanvasTexture(canvasPOS); texPOS.magFilter = THREE.NearestFilter;
let ekranMonitora = new THREE.Mesh(new THREE.PlaneGeometry(0.48, 0.24), new THREE.MeshBasicMaterial({ map:texPOS }));
ekranMonitora.position.set(1.5, 1.35, -0.22); scene.add(ekranMonitora);

// === STOLIK Z LAPTOPEM (CZARNY RYNEK) ===
// Stolik po lewej stronie gracza
retroBox(-4.2, 0.45, 1.8, 1.6, 0.9, 1.0, generujPixelTexture("standard", "#6e4020"));
retroBox(-4.2, 0.92, 1.8, 0.05, 0.05, 0.05, generujPixelTexture("standard", "#3a2210"));
// Baza laptopa
retroBox(-4.2, 0.93, 1.8, 0.7, 0.04, 0.5, generujPixelTexture("standard", "#1a1a1a"));
// Ekran laptopa
let laptopCanvas = document.createElement("canvas");
laptopCanvas.width = 256; laptopCanvas.height = 160;
let laptopCtx = laptopCanvas.getContext("2d");
let laptopTex = new THREE.CanvasTexture(laptopCanvas);
laptopTex.magFilter = THREE.NearestFilter; laptopTex.minFilter = THREE.NearestFilter;
let laptopRamka = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.45, 0.55), generujPixelTexture("standard", "#1a1a1a"));
laptopRamka.position.set(-3.88, 1.16, 1.78); scene.add(laptopRamka);
let laptopEkran = new THREE.Mesh(new THREE.PlaneGeometry(0.4, 0.42), new THREE.MeshBasicMaterial({ map: laptopTex }));
laptopEkran.rotation.y = Math.PI / 2;
laptopEkran.position.set(-3.86, 1.16, 1.78);
scene.add(laptopEkran);

function rysujLaptop() {
    laptopCtx.fillStyle = "#0a0014"; laptopCtx.fillRect(0,0,256,160);
    for(let i=0;i<5;i++){
        laptopCtx.fillStyle = "rgba(255,0,77,0.15)";
        laptopCtx.fillRect(0, Math.random()*160, 256, 1);
    }
    laptopCtx.fillStyle = "#ff004d"; laptopCtx.font = "bold 22px Courier New";
    laptopCtx.fillText("CZARNY", 60, 50);
    laptopCtx.fillText("RYNEK", 75, 78);
    laptopCtx.fillStyle = "#00e436"; laptopCtx.font = "bold 12px Courier New";
    laptopCtx.fillText("> burger.exe ready_", 30, 110);
    laptopCtx.fillStyle = "#ffec27"; laptopCtx.font = "bold 10px Courier New";
    laptopCtx.fillText("[A] otworz  [D] magazyn", 30, 140);
    laptopTex.needsUpdate = true;
}
rysujLaptop();
setInterval(rysujLaptop, 600);

// === MAGAZYN (za plecami gracza, kierunek +Z) ===
// Ściana z drzwiami
retroBox(0, 2, 5.0, 20, 5, 0.2, matSciana);
// Framuga drzwi
retroBox(-1.2, 1.5, 4.88, 0.2, 3.0, 0.05, generujPixelTexture("standard", "#3a2210"));
retroBox(1.2, 1.5, 4.88, 0.2, 3.0, 0.05, generujPixelTexture("standard", "#3a2210"));
retroBox(0, 3.0, 4.88, 2.6, 0.2, 0.05, generujPixelTexture("standard", "#3a2210"));
// Drzwi (ciemne, kanwa dla złodzieja)
let magCanvas = document.createElement("canvas");
magCanvas.width = 256; magCanvas.height = 256;
let magCtx = magCanvas.getContext("2d");
let magTex = new THREE.CanvasTexture(magCanvas);
magTex.magFilter = THREE.NearestFilter; magTex.minFilter = THREE.NearestFilter;
let magDrzwi = new THREE.Mesh(new THREE.PlaneGeometry(2.4, 2.8), new THREE.MeshBasicMaterial({ map: magTex }));
magDrzwi.position.set(0, 1.5, 4.87);
magDrzwi.rotation.y = Math.PI; // patrzy w stronę gracza
scene.add(magDrzwi);
// Napis MAGAZYN nad drzwiami
retroBox(0, 3.3, 4.85, 1.8, 0.4, 0.05, generujPixelTexture("standard", "#ffec27"));

let zlodziejObecny = false;
let zlodziejTimer = 0; // ile sekund został w drzwiach
let zlodziejGraceMs = 4000; // czas reakcji
let zlodziejStartTs = 0;

function rysujMagazyn() {
    magCtx.fillStyle = "#08060a"; magCtx.fillRect(0,0,256,256);
    // deski drzwi
    magCtx.fillStyle = "#2a1810";
    for(let y=0;y<256;y+=40){ magCtx.fillRect(0,y,256,2); }
    magCtx.fillStyle = "#1a0e08"; magCtx.fillRect(0,0,256,256);
    // klamka
    magCtx.fillStyle = "#888"; magCtx.fillRect(220, 130, 10, 18);
    // ciemny pasek otwartych drzwi
    magCtx.fillStyle = "#000"; magCtx.fillRect(40, 30, 180, 200);
    if(zlodziejObecny) {
        // sylwetka złodzieja
        magCtx.fillStyle = "#ff004d"; magCtx.fillRect(90, 50, 80, 8);
        magCtx.fillStyle = "#1a1a1a";
        magCtx.fillRect(95, 60, 70, 70); // tułów
        magCtx.fillRect(110, 35, 40, 35); // głowa
        magCtx.fillStyle = "#ff004d"; magCtx.fillRect(115, 48, 8, 6); magCtx.fillRect(140, 48, 8, 6); // oczy
        magCtx.fillStyle = "#ffec27"; magCtx.font = "bold 14px Courier New";
        magCtx.fillText("ZLODZIEJ!", 75, 200);
        magCtx.font = "bold 10px Courier New";
        let pozostalo = Math.max(0, zlodziejGraceMs - (Date.now() - zlodziejStartTs));
        magCtx.fillStyle = "#fff";
        magCtx.fillText("[SPACJA] PRZEGON  " + (pozostalo/1000).toFixed(1)+"s", 30, 225);
    } else {
        magCtx.fillStyle = "#444"; magCtx.font = "bold 10px Courier New";
        magCtx.fillText("magazyn - pusto", 70, 130);
    }
    magTex.needsUpdate = true;
}
rysujMagazyn();
setInterval(rysujMagazyn, 200);

// === SYSTEM OBROTU KAMERY (kasa <-> laptop <-> magazyn) ===
let widok = 'kasa';
const lookTargetKasa    = new THREE.Vector3(0, 1.6, -4);
const lookTargetLaptop  = new THREE.Vector3(-5, 1.15, 1.78);
const lookTargetMagazyn = new THREE.Vector3(0, 1.5, 5);
let currentLook = lookTargetKasa.clone();

function toggleCzarnyRynek(otwarty) {
    document.getElementById("black-market").style.display = otwarty ? "block" : "none";
}
function aktualizujHintObrot() {
    let h = document.getElementById("hint-obrot");
    if(!h) return;
    if(widok === 'kasa') h.innerText = "[A] laptop  |  [D] magazyn";
    else if(widok === 'laptop') h.innerText = "[D] wroc do kasy";
    else h.innerText = "[A] wroc do kasy  |  [SPACJA] przegon zlodzieja";
}

// === SYSTEM BUFFOW (TYMCZASOWE ULEPSZENIA Z BURGEROW) ===
let buffy = { tasmaSpeed: 1, zyskMnoznik: 1, nietykalny: false };
let buffTimers = {};

function pokazBuff(tekst) {
    document.getElementById("dymek").innerHTML = tekst;
    document.getElementById("dymek").style.borderColor = "#ff004d";
}

const BURGERY_DEF = {
    moc:   { koszt: 200, dzien: 4,  nazwa: "Burger Mocy",   akcja: () => {
        buffy.tasmaSpeed = 2;
        clearTimeout(buffTimers.moc);
        buffTimers.moc = setTimeout(() => { buffy.tasmaSpeed = 1; pokazBuff("⏱️ Burger Mocy wygasł."); }, 30000);
    }},
    big:   { koszt: 150, dzien: 8,  nazwa: "Big Energy",    akcja: () => {
        energiaPracownika = 100;
        document.getElementById("energy-bar-fill").style.width = "100%";
    }},
    zen:   { koszt: 100, dzien: 12, nazwa: "Burger Zen",    akcja: () => {
        cierpliwość = 150;
        document.getElementById("progress-bar-fill").style.width = "100%";
    }},
    combo: { koszt: 250, dzien: 16, nazwa: "Combo Burger",  akcja: () => {
        combo += 3;
        document.getElementById("val-combo").innerText = "x" + combo;
    }},
    zloty: { koszt: 400, dzien: 20, nazwa: "Złoty Burger",  akcja: () => {
        buffy.zyskMnoznik = 2;
        clearTimeout(buffTimers.zyskMnoznik);
        buffTimers.zyskMnoznik = setTimeout(() => { buffy.zyskMnoznik = 1; pokazBuff("💸 Złoty Burger wygasł."); }, 45000);
    }},
    haker: { koszt: 500, dzien: 24, nazwa: "Burger Hakera", akcja: () => {
        buffy.nietykalny = true;
        clearTimeout(buffTimers.nietykalny);
        buffTimers.nietykalny = setTimeout(() => { buffy.nietykalny = false; pokazBuff("🛡️ Burger Hakera wygasł."); }, 60000);
    }}
};

function odswiezBurgeryUI() {
    Object.keys(BURGERY_DEF).forEach(typ => {
        let el = document.querySelector(`.burger-item[data-typ="${typ}"]`);
        if(!el) return;
        let b = BURGERY_DEF[typ];
        let zablokowany = aktualnyDzien < b.dzien;
        el.classList.toggle("locked", zablokowany);
        let btn = el.querySelector(".burger-btn");
        let lockEl = el.querySelector(".b-lock");
        if(zablokowany) {
            btn.innerText = "🔒 DZIEŃ " + b.dzien;
            if(lockEl) lockEl.innerText = "Odblokowane od dnia " + b.dzien;
        } else {
            btn.innerText = "KUP";
            if(lockEl) lockEl.innerText = "";
        }
    });
}

function kupBurger(typ) {
    const b = BURGERY_DEF[typ];
    if(!b) return;
    if(aktualnyDzien < b.dzien) {
        playSound(120, 0.25, 'sawtooth');
        pokazBuff(`🔒 ${b.nazwa} odblokowany od DNIA ${b.dzien}!`);
        return;
    }
    if(zarobek < b.koszt) { playSound(150, 0.2, 'sawtooth'); return; }
    zarobek -= b.koszt;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    b.akcja();
    playSound(700, 0.15, 'sine');
    pokazBuff(`🍔 Zjadłeś: <b>${b.nazwa}</b>! Buff aktywny.`);
}

function kupOpinie() {
    if(gwiazdki >= maxGwiazdki) { playSound(150, 0.2, 'sawtooth'); pokazBuff("⭐ Masz już maksymalną opinię!"); return; }
    if(zarobek < 1000) { playSound(150, 0.2, 'sawtooth'); pokazBuff("💸 Brakuje kasy! Potrzeba 1000 zł."); return; }
    zarobek -= 1000;
    gwiazdki++;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
    document.getElementById("val-gwiazdki").innerText = "⭐".repeat(Math.max(0, gwiazdki)) + "❌".repeat(Math.max(0, maxGwiazdki-gwiazdki));
    playSound(800, 0.2, 'sine');
    pokazBuff("⭐ Kupiłeś lewą opinię! +1 gwiazdka");
}

// === SYSTEM ZŁODZIEJA W MAGAZYNIE ===
function probaZlodzieja() {
    if(!graAktywna) return;
    if(zlodziejObecny) return;
    if(buffy.nietykalny) return;
    if(Math.random() < 0.07) {
        zlodziejObecny = true;
        zlodziejStartTs = Date.now();
        playSound(180, 0.3, 'sawtooth');
        if(widok !== 'magazyn') {
            pokazBuff("⚠️ Hałas w MAGAZYNIE! Naciśnij [D]!");
        }
        // jeśli gracz nie zareaguje, kradzież
        setTimeout(() => {
            if(zlodziejObecny) {
                zlodziejObecny = false;
                zarobek = Math.max(0, zarobek - 300);
                document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
                playSound(90, 0.5, 'sawtooth');
                pokazBuff("💸 Złodziej ukradł towar! -300 zł");
                document.getElementById("dymek").style.borderColor = "#ff004d";
            }
        }, zlodziejGraceMs);
    }
}
setInterval(probaZlodzieja, 8000);

function przegonZlodzieja() {
    if(!zlodziejObecny) return;
    if(widok !== 'magazyn') return;
    zlodziejObecny = false;
    playSound(800, 0.2, 'square');
    pokazBuff("🛡️ Przegoniłeś złodzieja!");
}


function pokazKase(){
    ctxPOS.fillStyle = "#112211"; ctxPOS.fillRect(0, 0, 256, 128);
    ctxPOS.fillStyle = "#00ff44"; ctxPOS.font = "bold 15px Courier New";
    
    if(energiaPracownika <= 20) {
        if(blink) { ctxPOS.fillStyle = "#3a0000"; ctxPOS.fillRect(0,0,256,128); }
        ctxPOS.fillStyle = "#ff5555"; ctxPOS.fillText("⚠️ SŁABA ENERGIA!", 15, 20);
        ctxPOS.fillStyle = "#00ff44";
    }

    if(aktywneZadanieZaplecza) {
        ctxPOS.fillStyle = "#ff0077"; ctxPOS.fillRect(0,0,256,128);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("📋 PILNY TASK KORPO!", 15, 45);
        ctxPOS.fillText("IDŹ DO ZAKŁADKI TASKI", 15, 75);
        texPOS.needsUpdate = true; return;
    }
    if(stanKasyAwaria) {
        ctxPOS.fillStyle = "#ff0000"; ctxPOS.fillRect(0,0,256,128);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("🚨 AWARIA SYSTEMU!", 15, 45);
        ctxPOS.fillText("KLIKAJ [R] ("+awariaKlikniecia+"/10)", 15, 75);
        texPOS.needsUpdate = true; return;
    }
    if(stanBhpSzalenstwo) {
        ctxPOS.fillStyle = "#ffaa00"; ctxPOS.fillText("🚨 REWIZJA BHP!", 15, 45);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("TAŚMA NIESTABILNA!", 15, 75);
        texPOS.needsUpdate = true; return;
    }                                                                                                                                                 
    
    // NAPRAWIONY BŁĄD SKŁADNIOWY TUTAJ:
    if(uciekayacyZlodziej) {
        ctxPOS.fillStyle = "#ff004d"; ctxPOS.fillRect(0,0,256,128);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("🦹 ZŁODZIEJ UCIEKA!", 15, 55);
        ctxPOS.fillText("WCIŚNIJ [G]!!!", 15, 85);
        texPOS.needsUpdate = true; return;
    }

    if(wymagaWeryfikacjiWieku && !zweryfikowanoWiek) {
        ctxPOS.fillStyle = "#000"; ctxPOS.fillRect(0,0,256,128); ctxPOS.fillStyle = "#ffec27";
        ctxPOS.fillText("🪪 WERYFIKACJA 18+", 15, 35);
        ctxPOS.fillStyle = "#fff"; ctxPOS.fillText("Wiek klienta: " + wiekKlienta, 15, 65);
        ctxPOS.fillText("[Y] Zezwól | [N] Odmów", 15, 95);
        texPOS.needsUpdate = true; return;
    }

    if(wymagaAplikacji && !zatwierdzonoAplikacje) {
        ctxPOS.fillStyle = "#29adff"; ctxPOS.fillText("📱 PROGRAM LOJALNOŚCI", 15, 35);
        ctxPOS.fillStyle = "#fff"; ctxPOS.fillText("KOD: " + kodAplikacjiWymagany, 15, 65);
        ctxPOS.fillStyle = "#00ff44"; ctxPOS.fillText("WPISZ & [P]: " + kodAplikacjiWpisany + (blink ? "_" : ""), 15, 95);
        texPOS.needsUpdate = true; return;
    }

    if(produktZablokowany) {
        ctxPOS.fillStyle = "#ffec27"; ctxPOS.fillText("⚠️ KOD USZKODZONY!", 15, 35);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("WPISZ CENE Z DYMKA", 15, 65);
        ctxPOS.fillStyle = "#00ff44"; ctxPOS.fillText("CENA: " + wpis + (blink ? "_" : ""), 15, 95);
        texPOS.needsUpdate = true; return;
    }

    if(wymagaSprzatania) {
        ctxPOS.fillStyle = "#83769c"; ctxPOS.fillRect(0,0,256,128);
        ctxPOS.fillStyle = "#ffffff"; ctxPOS.fillText("🧹 ROZLANY PŁYN!", 15, 55);
        ctxPOS.fillText("WCIŚNIJ [C] ABY UMYĆ", 15, 85);
        texPOS.needsUpdate = true; return;
    }

    let yOffset = (energiaPracownika <= 20) ? 15 : 0;
    ctxPOS.fillText("RACHUNEK: " + cena.toFixed(2) + " ZŁ", 15, 35 + yOffset);

    if(typPlatnosci === "gotowka"){
        ctxPOS.fillStyle = "#ffff00"; ctxPOS.fillText("KLIENT DAŁ: " + dal.toFixed(2), 15, 65 + yOffset);
        ctxPOS.fillStyle = "#00ff44"; ctxPOS.fillText("WYDAJ: " + wpis + (blink ? "_" : ""), 15, 95 + yOffset);
    } else {
        ctxPOS.fillStyle = "#ff77a8"; ctxPOS.fillText("PŁATNOŚĆ KARTĄ", 15, 65 + yOffset);
        ctxPOS.fillStyle = "#00ff44"; ctxPOS.fillText(ulepszenia.dotykowyPos ? "ENTER (AUTO)" : "KLIKNIJ 0 + ENTER", 15, 95 + yOffset);
    }
    texPOS.needsUpdate = true;
}

setInterval(() => { blink = !blink; pokazKase(); }, 350);

function generujTwarzMaterial(kolorSkory) {
    let canvas = document.createElement("canvas"); canvas.width = 32; canvas.height = 32;
    let ctx = canvas.getContext("2d"); ctx.imageSmoothingEnabled = false;
    ctx.fillStyle = kolorSkory; ctx.fillRect(0,0,32,32);
    // brwi
    ctx.fillStyle = "#000";
    ctx.fillRect(5,7,8,1); ctx.fillRect(4,8,2,1); ctx.fillRect(12,8,2,1);
    ctx.fillRect(19,7,8,1); ctx.fillRect(18,8,2,1); ctx.fillRect(26,8,2,1);
    // białka oczu
    ctx.fillStyle = "#fff";
    ctx.fillRect(5,11,8,5); ctx.fillRect(19,11,8,5);
    // obwódka oczu
    ctx.fillStyle = "#000";
    ctx.fillRect(5,10,8,1); ctx.fillRect(5,16,8,1); ctx.fillRect(4,11,1,5); ctx.fillRect(13,11,1,5);
    ctx.fillRect(19,10,8,1); ctx.fillRect(19,16,8,1); ctx.fillRect(18,11,1,5); ctx.fillRect(27,11,1,5);
    // tęczówki granatowe
    ctx.fillStyle = "#1d2b7a";
    ctx.fillRect(7,12,5,3); ctx.fillRect(21,12,5,3);
    // refleks cyjan
    ctx.fillStyle = "#29adff";
    ctx.fillRect(8,13,2,1); ctx.fillRect(22,13,2,1);
    // nos
    ctx.fillStyle = "#000";
    ctx.fillRect(16,14,1,7); ctx.fillRect(15,21,3,1);
    // usta uśmiech
    ctx.fillStyle = "#000";
    ctx.fillRect(10,25,12,1); ctx.fillRect(9,26,1,1); ctx.fillRect(22,26,1,1); ctx.fillRect(10,27,12,1);
    ctx.fillStyle = "#c97e4a";
    ctx.fillRect(10,26,12,1);

    let tex = new THREE.CanvasTexture(canvas);
    tex.magFilter = THREE.NearestFilter; tex.minFilter = THREE.NearestFilter;
    return new THREE.MeshStandardMaterial({ map: tex, roughness: 1.0 });
}

function stworzPostacKlienta(indexKolejki, isZloty = false) {
    let gPostac = new THREE.Group();
    let kolorSkory = "#e99a75";
    let matSkora = generujPixelTexture("standard", kolorSkory);
    let matTwarz = generujTwarzMaterial(kolorSkory);
    let kolorUbrania = isZloty ? "#ffec27" : ["#ff004d", "#29adff", "#00e436", "#83769c"][Math.floor(Math.random()*4)];
    let matUbranie = generujPixelTexture("standard", kolorUbrania);
    
    let t_body = new THREE.Mesh(new THREE.BoxGeometry(0.7, 1.1, 0.4), matUbranie); t_body.position.set(0, 0.55, 0); gPostac.add(t_body);
    // Kolejność materiałów BoxGeometry: +X,-X,+Y,-Y,+Z(przód),-Z
    let matyGlowa = [matSkora, matSkora, matSkora, matSkora, matTwarz, matSkora];
    let t_head = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.4), matyGlowa); t_head.position.set(0, 1.35, 0); gPostac.add(t_head);
    scene.add(gPostac); gPostac.position.set(0, 0.4, -1.5 - (indexKolejki * 1.5));
    return gPostac;
}

function odswiezKolejke() {
    kolejkaKlientow.forEach(p => scene.remove(p)); kolejkaKlientow = [];
    let ile = Math.min(4, KLIENTOW_NA_DZIEN - klienciWDniu);
    for(let i=0; i<ile; i++) {
        let zloty = (i === 0 && jestZlotyKlient);
        kolejkaKlientow.push(stworzPostacKlienta(i, zloty));
    }
}

function nowyKlient(){
    if(!graAktywna) return;
    if(ulepszenia.klon && aktualnyDzien % 2 === 0 && klienciWDniu === 4) {
        klienciWDniu += 2; document.getElementById("dymek").innerHTML = "👥 Klon obsłużył 2 klientów za Ciebie!";
    }
    if(klienciWDniu >= KLIENTOW_NA_DZIEN) { pokazPodsumowanieDnia(); return; }
     
    jestZlotyKlient = (Math.random() < 0.15); 
    wymagaSprzatania = (trybGry === "latwy") ? false : (Math.random() < 0.12); 
    
    odswiezKolejke();
    cubes.forEach(x => scene.remove(x)); cubes = []; cena = 0; dal = 0; wpis = ""; produktZablokowany = null;
    wymagaWeryfikacjiWieku = false; zweryfikowanoWiek = false; 
    wymagaAplikacji = false; kodAplikacjiWpisany = ""; zatwierdzonoAplikacje = false;
    uciekayacyZlodziej = false; zlodziejZlapany = false;
    
    let bazowaCierpliwosc = (trybGry === "latwy") ? 180 : 100;
    cierpliwość = ulepszenia.buty ? bazowaCierpliwosc + 10 : bazowaCierpliwosc; czasKlienta = 0;
    if(ulepszenia.roslinka) cierpliwość += 15;
    
    document.getElementById("progress-bar-fill").style.width = "100%";
    typPlatnosci = Math.random() > 0.4 ? "gotowka" : "karta";
    spieszySie = Math.random() < 0.25 || jestZlotyKlient; 

    let randZdarzenie = Math.random();
    if(!buffy.nietykalny) {
        if(randZdarzenie < 0.08 && wynik > 2) {
            stanKasyAwaria = true; awariaKlikniecia = 0; playSound(120, 0.5, 'sawtooth');
        } else if(randZdarzenie > 0.08 && randZdarzenie < 0.15 && wynik > 3) {
            stanBhpSzalenstwo = true; bhpMnoznikPredkosci = Math.random() > 0.5 ? 2.5 : 0.3;
            setTimeout(() => { stanBhpSzalenstwo = false; bhpMnoznikPredkosci = 1.0; ustawDymekStatusu(); }, 10000);
        }
        if(Math.random() < 0.08 && wynik > 4) {
            if(trybGry !== "latwy") uciekayacyZlodziej = true;
            zlodziejCzasNaReakcje = Date.now();
        }
    }

    if(Math.random() < 0.30 && !uciekayacyZlodziej) {
        wymagaAplikacji = true;
        let znaki = "BCEFGHIJKLMNOQRSTUVWXYZ"; 
        kodAplikacjiWymagany = znaki[Math.floor(Math.random()*znaki.length)] + znaki[Math.floor(Math.random()*znaki.length)] + znaki[Math.floor(Math.random()*znaki.length)];
    }

    let ile = 2 + Math.floor(Math.random() * 3);
    for(let i=0; i<ile; i++){
        let p = produkty[Math.floor(Math.random()*produkty.length)];
        let h = 0.25; let mat = generujPixelTexture(p[2], p[3]);
        let cube = new THREE.Mesh(new THREE.BoxGeometry(0.22, h, 0.22), mat);
        cube.position.set(-1.6 - (i * 0.42), 1.02 + (h/2), -0.3);
        
        let kodUszkodzony = Math.random() < 0.15;
        cube.userData = { dane: p, zeskanowany: false, kodUszkodzony: kodUszkodzony };
        
        if (p[4] && Math.random() < 0.4) {
            wymagaWeryfikacjiWieku = true;
            wiekKlienta = Math.random() > 0.4 ? (18 + Math.floor(Math.random()*30)) : (14 + Math.floor(Math.random()*4));
        }
        if(ulepszenia.qrSkaner && i === 0 && wynik % 5 === 0 && !kodUszkodzony) {
            cube.userData.zeskanowany = true;
            let koszt = aktualneCeny[p[0]]; if(ulepszenia.monopol) koszt *= 1.5; if(ulepszenia.plakat) koszt += 1;
            cena += koszt; cube.material.color.setHex(0x00e436);
        }
        scene.add(cube); cubes.push(cube);
    }
    
    // ZMIENIONO WARUNEK: Uruchamia zadanie zaplecza co 10 klientów
    if(licznikDoTaska >= 10) {
        aktywujZadanieZaplecza();
    } else {
        ustawDymekStatusu();
        pokazKase();
    }

    clearInterval(cierpliwośćTimer);
    cierpliwośćTimer = setInterval(() => {
        if(!graAktywna || aktywneZadanieZaplecza) return;
        czasKlienta += 0.35;
        if(ulepszenia.klima && czasKlienta < 3.0) return;
        if(ulepszenia.glosnik && Math.random() > 0.96 && cierpliwość < 70) { cierpliwość += 20; playSound(600, 0.1, 'sine'); }

        let spadek = ulepszenia.melisa ? 0.7 : 1.8;
        if(trybGry === "latwy") spadek *= 0.5;
        if(spieszySie) spadek *= 2.0;
        if(wymagaSprzatania) spadek *= 2.5; 

        cierpliwość -= spadek;
        document.getElementById("progress-bar-fill").style.width = Math.min(100, Math.max(0, cierpliwość)) + "%";
        
        if(uciekayacyZlodziej && !zlodziejZlapany) {
            if(kolejkaKlientow[0]) {
                kolejkaKlientow[0].position.z -= 0.15; 
                if(Date.now() - zlodziejCzasNaReakcje > 2500) { 
                    clearInterval(cierpliwośćTimer);
                    uciekayacyZlodziej = false;
                    obsluzBladCombo();
                    let kara = 150; zarobek -= kara;
                    odejmijGwiazdke(`😡 Złodziej uciekł z towarami! Kara od managera: -${kara} zł`);
                    klienciWDniu++;
                    setTimeout(nowyKlient, 1500);
                    return;
                }
            }
        }

        if(cierpliwość <= 0) {
            clearInterval(cierpliwośćTimer); obsluzBladCombo(); odejmijGwiazdke("😡 Ile można czekać!");
            klienciWDniu++; document.getElementById("val-do-konca").innerText = KLIENTOW_NA_DZIEN - klienciWDniu;
            setTimeout(nowyKlient, 1200);
        }
    }, 350);
}

function ustawDymekStatusu() {
    let dym = document.getElementById("dymek"); dym.style.borderColor = "#47af4c";
    
    if(aktywneZadanieZaplecza) {
        dym.innerHTML = "📋 <b>BLOKADA: OBOWIĄZKOWE ZADANIE KORPO!</b> Manager każe iść na zaplecze! Wejdź w zakładkę <b>TASKI KORPO</b>.";
        dym.style.borderColor = "#ff77a8";
    } else if(uciekayacyZlodziej) {
        dym.innerHTML = "🦹 <b>ZŁODZIEJ CHWYCIŁ TOWAR I UCIEKA!</b> Wciśnij szybko <b>[G]</b>, aby wezwać ochronę!";
        dym.style.borderColor = "#ff004d";
    } else if(stanKasyAwaria) {
        dym.innerHTML = "🚨 SYSTEM ZAWIESZONY! Napraw kasę klikając [R]!";
        dym.style.borderColor = "#ff004d";
    } else if(wymagaSprzatania) {
        dym.innerHTML = "🧹 <b>KTOŚ ROZLAŁ SOK!</b> Klienci się brzydzą! Szybko wciśnij <b>[C]</b> aby umyć podłogę!";
        dym.style.borderColor = "#83769c";
    } else if(stanBhpSzalenstwo) {
        dym.innerHTML = "🚨 <b>INSPEKCJA BHP!</b> Silnik taśmy przegrzewa się, zachowaj ostrożność!";
        dym.style.borderColor = "#ffaa00";
    } else if(wymagaWeryfikacjiWieku && !zweryfikowanoWiek) {
        dym.innerHTML = `🪪 Klient kupuje towar 18+. Deklarowany wiek: <b>${wiekKlienta} lat</b>. Kliknij <b>[Y]</b> (Weryfikacja OK) lub <b>[N]</b> (Niepełnoletni)!`;
        dym.style.borderColor = "#ffec27";
    } else if(wymagaAplikacji && !zatwierdzonoAplikacje) {
        dym.innerHTML = `📱 Klient prosi o aplikację korporacyjną. Przepisz kod: <b style='color:#00e436'>${kodAplikacjiWymagany}</b> i kliknij <b>[P]</b>!`;
        dym.style.borderColor = "#29adff";
    } else if(produktZablokowany) {
        let kosztZatarty = aktualneCeny[produktZablokowany.userData.dane[0]];
        if(ulepszenia.monopol) kosztZatarty *= 1.5; if(ulepszenia.plakat) kosztZatarty += 1;
        dym.innerHTML = `⚠️ Kod zatarty! Wpisz aktualną cenę z cennika: <b>${kosztZatarty} zł</b> i zatwierdź klikając <b>[E]</b>!`;
        dym.style.borderColor = "#ffec27";
    } else if(jestZlotyKlient) {
        dym.innerHTML = "👑 <b>ZŁOTY KLIENT VIP!</b> Bardzo się spieszy, ale zostawi ogromny napiwek!";
        dym.style.borderColor = "#ffec27";
    } else if(spieszySie) {
        dym.innerHTML = "🏃 Śpieszę się na autobus! Szybciej! (Cierpliwość spada x2)";
        dym.style.borderColor = "#ffaa00";
    } else {
        dym.innerHTML = "🙂 Proszę skasować.";
    }
}

function obsluzBladCombo() {
    if(ulepszenia.zloteCombo && combo > 1) { combo = Math.max(1, Math.floor(combo / 2)); } else { combo = 1; }
    document.getElementById("val-combo").innerText = "x" + combo;
}

function odejmijGwiazdke(powod) {
    gwiazdki--; playSound(140, 0.3, 'square');
    document.getElementById("dymek").innerHTML = powod; document.getElementById("dymek").style.borderColor = "#ff004d";
    document.getElementById("val-gwiazdki").innerText = "⭐".repeat(Math.max(0, gwiazdki)) + "❌".repeat(Math.max(0, maxGwiazdki-gwiazdki));
    if(gwiazdki <= 0) { 
        graAktywna = false; 
        document.getElementById("game-over-reason").innerText = "Zwolniony za złą opinię klientów lub łamanie procedur!";
        document.getElementById("game-over-screen").style.display = "flex"; 
    }
}

function pokazPodsumowanieDnia() {
    clearInterval(cierpliwośćTimer); tasmaPracuje = false;
    if(ulepszenia.prezes) { zarobek += utargWDniu; }
    
    let podsumowanietekst = `<b>RAPORT Z KASY (DZIEŃ ${aktualnyDzien})</b><br><br>🛒 Obsłużeni: ${klienciWDniu}<br>💰 Utarg: +${utargWDniu.toFixed(2)} zł`;
    
    if(aktywneKredyty > 0) {
        let kosztKredytu = aktywneKredyty * 150; zarobek -= kosztKredytu;
        podsumowanietekst += `<br><span style='color:#ff004d'>🚨 Odciągnięto ratę kredytów: -${kosztKredytu} zł</span>`;
    }

    if(aktualnyDzien % 3 === 0) {
        let kosztRachunkow = 300 + (aktualnyDzien * 50); zarobek -= kosztRachunkow;
        podsumowanietekst += `<br><span style='color:#ff9f77'>🧾 Opłacono koszty stanowiska i prądu: -${kosztRachunkow} zł</span>`;
    }

    podsumowanietekst += `<br><br><b>Stan Konta po potrąceniach: ${zarobek.toFixed(2)} zł</b>`;
    document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";

    if(zarobek < 0) {
        graAktywna = false;
        document.getElementById("game-over-reason").innerHTML = `Firma zbankrutowała! Wygenerowałeś debet: ${zarobek.toFixed(2)} zł. System odcina dostęp!`;
        document.getElementById("game-over-screen").style.display = "flex";
        return;
    }

    document.getElementById("day-summary-stats").innerHTML = podsumowanietekst;
    document.getElementById("day-clear-screen").style.display = "flex";
}

function nastepnyDzien() {
    aktualnyDzien++; klienciWDniu = 0; utargWDniu = 0;
    document.getElementById("val-dzien-etykieta").innerText = "DZIEŃ " + aktualnyDzien;
    document.getElementById("val-do-konca").innerText = KLIENTOW_NA_DZIEN;
    
    przeliczInflacje();

    let dniDoRachunku = 3 - (aktualnyDzien % 3);
    if(dniDoRachunku === 3) dniDoRachunku = 0; 
    document.getElementById("val-rachunki-licznik").innerText = (dniDoRachunku === 0 ? "DZISIAJ" : dniDoRachunku + " dni");
    
    odswiezBurgeryUI();
    document.getElementById("day-clear-screen").style.display = "none";
    nowyKlient();
}

////////////////////////////////////
// STEROWANIE / INPUT SYSTEM
window.onkeydown = e => {
    // Obrót kamery: A = lewo (laptop), D = prawo (magazyn)
    if(graAktywna) {
        let k = e.key.toLowerCase();
        if(k === 'a') {
            if(widok === 'kasa')        { widok = 'laptop';  toggleCzarnyRynek(true);  playSound(400, 0.12, 'triangle'); aktualizujHintObrot(); return; }
            if(widok === 'magazyn')     { widok = 'kasa';    playSound(500, 0.12, 'triangle'); aktualizujHintObrot(); return; }
        }
        if(k === 'd') {
            if(widok === 'kasa')        { widok = 'magazyn'; playSound(400, 0.12, 'triangle'); aktualizujHintObrot(); return; }
            if(widok === 'laptop')      { widok = 'kasa';    toggleCzarnyRynek(false); playSound(500, 0.12, 'triangle'); aktualizujHintObrot(); return; }
        }
        if(k === ' ' && widok === 'magazyn') { przegonZlodzieja(); e.preventDefault(); return; }
    }
    // Gdy nie patrzymy na kasę, klawiatura kasy zamrożona
    if(widok !== 'kasa') return;


    if(!graAktywna || klienciWDniu >= KLIENTOW_NA_DZIEN) return;
    
    // Blokada klawiatury kasy na czas aktywnego zadania zaplecza
    if(aktywneZadanieZaplecza) return;

    if(wymagaSprzatania) {
        if(e.key.toLowerCase() === 'c') {
            wymagaSprzatania = false;
            playSound(400, 0.1, 'triangle');
            ustawDymekStatusu(); pokazKase();
        }
        return; 
    }

    if(uciekayacyZlodziej) {
        if(e.key.toLowerCase() === 'g') {
            zlodziejZlapany = true; uciekayacyZlodziej = false;
            let nagroda = 200; zarobek += nagroda; utargWDniu += nagroda;
            playSound(900, 0.3, 'sine');
            document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
            document.getElementById("dymek").innerHTML = `🦹 Ochrona ujęła złodzieja! Premia od firmy: +${nagroda} zł!`;
            setTimeout(nowyKlient, 1500);
        }
        return;
    }

    if(stanKasyAwaria) {
        if(e.key.toLowerCase() === 'r') {
            awariaKlikniecia++; playSound(700, 0.03, 'sine');
            if(awariaKlikniecia >= 10) {
                stanKasyAwaria = false; playSound(950, 0.1, 'sine');
                ustawDymekStatusu(); pokazKase();
            }
        }
        return;
    }

    if(wymagaWeryfikacjiWieku && !zweryfikowanoWiek) {
        let k = e.key.toLowerCase();
        if(k === 'y') {
            if(wiekKlienta >= 18) {
                zweryfikowanoWiek = true; playSound(800, 0.1, 'sine'); ustawDymekStatusu(); pokazKase();
            } else {
                obsluzBladCombo(); zarobek -= 100;
                odejmijGwiazdke("😡 Sprzedałeś towar nieletniemu! Kara sanepidu: -100 zł!");
                zweryfikowanoWiek = true; ustawDymekStatusu(); pokazKase();
            }
        } else if(k === 'n') {
            if(wiekKlienta < 18) {
                document.getElementById("dymek").innerHTML = "🚫 Prawidłowa odmowa sprzedaży! Klient odchodzi.";
                playSound(900, 0.1, 'sine'); klienciWDniu++; 
                licznikDoTaska++;
                renderujTaskUI();
                setTimeout(nowyKlient, 1200);
            } else {
                obsluzBladCombo();
                odejmijGwiazdke("😡 Klient miał dowód! Niesłusznie odmówiłeś sprzedaży!");
                klienciWDniu++; 
                licznikDoTaska++;
                renderujTaskUI();
                setTimeout(nowyKlient, 1200);
            }
        }
        return;
    }

    if(wymagaAplikacji && !zatwierdzonoAplikacje) {
        if( /[A-Za-z]/.test(e.key) && e.key.length === 1 && e.key.toLowerCase() !== "p" && kodAplikacjiWpisany.length < 3 ){
            kodAplikacjiWpisany += e.key.toUpperCase();
            pokazKase();
        }

        if(e.key === "Backspace"){
            kodAplikacjiWpisany = kodAplikacjiWpisany.slice(0,-1);
            pokazKase();
        }

        if(e.key.toLowerCase() === "p"){
            if(kodAplikacjiWpisany === kodAplikacjiWymagany){
                zatwierdzonoAplikacje=true;
                playSound(850,0.1,'sine');
                document.getElementById("dymek").innerHTML = "📱 Aplikacja zaakceptowana";
                ustawDymekStatusu(); pokazKase();
            }else{
                kodAplikacjiWpisany="";
                playSound(150,0.2,'sawtooth');
                document.getElementById("dymek").innerHTML = "❌ Zły kod";
                pokazKase();
            }
        }
        return;
    }

    if(produktZablokowany) {
        if(/\d/.test(e.key) && wpis.length < 5){ wpis += e.key; pokazKase(); }
        if(e.key == "Backspace"){ wpis = wpis.slice(0,-1); pokazKase(); }
        if(e.key.toLowerCase() === 'e') {
            let wpisanaCena = Number(wpis);
            let poprawnaCena = aktualneCeny[produktZablokowany.userData.dane[0]];
            if(ulepszenia.monopol) poprawnaCena *= 1.5; if(ulepszenia.plakat) poprawnaCena += 1;
            poprawnaCena = Number(poprawnaCena.toFixed(2));

            if(wpisanaCena == poprawnaCena) {
                produktZablokowany.userData.zeskanowany = true; cena += poprawnaCena;
                playSound(1100, 0.06, 'sine'); 
                produktZablokowany.material.color.setHex(0x00e436); produktZablokowany = null; wpis = "";
                
                let nieskanowane = cubes.filter(c => !c.userData.zeskanowany);
                if(nieskanowane.length === 0) { zamknijRachunekKlienta(); } else { ustawDymekStatusu(); pokazKase(); }
            } else {
                wpis = ""; playSound(150, 0.2, 'sawtooth');
                odejmijGwiazdke("😡 Wpisałeś złą cenę! Sprawdź cennik inflacyjny!"); pokazKase();
            }
        }
        return;
    }

    if(e.code === "Space" && !ulepszenia.autoTasma) { tasmaPracuje = true; return; }

    let nieskanowane = cubes.filter(c => !c.userData.zeskanowany); if(nieskanowane.length > 0) return;

    if(/\d/.test(e.key) && wpis.length < 5){ wpis += e.key; pokazKase(); }
    if(e.key == "Backspace"){ wpis = wpis.slice(0,-1); pokazKase(); }
    if(e.key == "Enter"){
        if(e.repeat) return;
        if(window._obslugaWTrakcie) return;
        if(wpis === "" && typPlatnosci === "gotowka") return;
        if(typPlatnosci === "karta" && ulepszenia.dotykowyPos && wpis === "") { wpis = "0"; }

        if(wymagaAplikacji && !zatwierdzonoAplikacje) {
            obsluzBladCombo(); 
            odejmijGwiazdke("😡 Zapomniałeś nabić aplikacji lojalnościowej klienta!");
            return;
        }

        let rWpisana = Number(wpis); 
        let rPoprawna = Number(reszta.toFixed(2)); 
        let poprawnie = (rWpisana == rPoprawna) || (ulepszenia.magnes && Math.abs(rWpisana - rPoprawna) <= 1);

        window._obslugaWTrakcie = true;

        if(poprawnie){
            wynik++; klienciWDniu++;
            licznikDoTaska++;
            
            let mnoznikKawy = ulepszenia.kawa ? 2 : 1;
            let bonusVip = (typPlatnosci === "karta" && ulepszenia.kartaVip) ? 100 : 0;
            let mnoznikZlotego = jestZlotyKlient ? 5 : 1; 
            
            let zarobione = (cena * combo * mnoznikKawy * mnoznikZlotego * mnoznikPrestizu * mnoznikKariery * buffy.zyskMnoznik) + bonusVip;                 
            zarobek += zarobione; utargWDniu += zarobione; combo++; playSound(800, 0.05, 'sine');
            
            energiaPracownika = Math.max(0, energiaPracownika - (10 - (ulepszenia.buty ? 3 : 0)));
            document.getElementById("energy-bar-fill").style.width = energiaPracownika + "%";

            if(jestZlotyKlient) document.getElementById("dymek").innerHTML = "👑 Złoty klient dał potężny napiwek!"; 
            else if(bonusVip > 0) document.getElementById("dymek").innerHTML = "💳 VIP dał +100 zł napiwku!"; 
            else document.getElementById("dymek").innerHTML = "🙂 Dziękuję!";
        } else {
            obsluzBladCombo(); klienciWDniu++; licznikDoTaska++; odejmijGwiazdke("😡 Błędna reszta!");
        }
        document.getElementById("val-wynik").innerText = wynik;
        document.getElementById("val-combo").innerText = "x" + combo; document.getElementById("val-zarobek").innerText = zarobek.toFixed(2) + " zł";
        document.getElementById("val-do-konca").innerText = KLIENTOW_NA_DZIEN - klienciWDniu;
        
        renderujTaskUI();
        setTimeout(() => { window._obslugaWTrakcie = false; nowyKlient(); }, 1200);
    }
};

window.onkeyup = e => { if(e.code === "Space") { tasmaPracuje = false; } };

function zamknijRachunekKlienta() {
    clearInterval(cierpliwośćTimer);
    if(typPlatnosci === "gotowka") {
        dal = cena + Math.floor(Math.random() * 20); if(dal < cena) dal = cena;
        reszta = dal - cena; if(ulepszenia.aiHelper) wpis = reszta.toFixed(2);
    } else { 
        reszta = 0; if(ulepszenia.aiHelper) wpis = "0";
    }
    ustawDymekStatusu(); pokazKase();
}

////////////////////////////////////
// GŁÓWNA PĘTLA GRAFICZNA
function anim(){
    requestAnimationFrame(anim);

    // Płynny obrót kamery między kasą a laptopem
    let target = (widok === 'laptop') ? lookTargetLaptop : (widok === 'magazyn' ? lookTargetMagazyn : lookTargetKasa);
    currentLook.lerp(target, 0.12);
    camera.lookAt(currentLook);

    for(let i = 0; i < kolejkaKlientow.length; i++) {
        let doceloweZ = -1.5 - (i * 1.5);
        if(uciekayacyZlodziej && i === 0) continue;
        if(kolejkaKlientow[i].position.z < doceloweZ) kolejkaKlientow[i].position.z += 0.08;
    }

    if((tasmaPracuje || ulepszenia.autoTasma) && graAktywna && !aktywneZadanieZaplecza && !stanKasyAwaria && !produktZablokowany && !uciekayacyZlodziej && !wymagaSprzatania) {
        let mnoznikiTasmy = { 1: 1, 2: 1.8, 3: 3.2, 4: 5.5 };
        let spd = 0.025 * mnoznikiTasmy[ulepszenia.tasmaLvl] * bhpMnoznikPredkosci * buffy.tasmaSpeed;
        
        cubes.forEach(cube => {
            if(cube.position.x < 0.8) {
                cube.position.x += spd;
                let bonusSwiatla = ulepszenia.oswietlenie ? 0.1 : 0.0;
                let graniceLasera = { 1: 0.72, 2: 0.62, 3: 0.40, 4: -0.10 };
                let granicaSanu = graniceLasera[ulepszenia.laserLvl] - bonusSwiatla;
                
                if(!cube.userData.zeskanowany && cube.position.x >= granicaSanu) {
                    
                    if(cube.userData.kodUszkodzony) {
                        produktZablokowany = cube; tasmaPracuje = false; wpis = "";
                        ustawDymekStatusu(); pokazKase(); playSound(300, 0.2, 'triangle');
                        return;
                    }

                    cube.userData.zeskanowany = true;
                    let kosztProduktu = aktualneCeny[cube.userData.dane[0]];
                    if(ulepszenia.monopol) kosztProduktu *= 1.5; if(ulepszenia.plakat) kosztProduktu += 1;
                    
                    cena += kosztProduktu; playSound(1100, 0.06, 'sine'); 
                    cube.material.color.setHex(0x00e436); pokazKase();

                    let nieskanowane = cubes.filter(c => !c.userData.zeskanowany);
                    if(nieskanowane.length === 0) { zamknijRachunekKlienta(); }
                }
            } else {
                let predkoscZrzutu = ulepszenia.szybkieRece ? 0.18 : 0.06;
                if(cube.position.y > 0.6) { cube.position.y -= predkoscZrzutu; cube.position.x += 0.02; }
            }
        });
    }
    renderer.render(scene, camera);
}
anim();

// FUNKCJA URUCHAMIAJĄCA GRĘ PO KLIKNIĘCIU "GRAJ"
function uruchomGre(tryb) {
    trybGry = (tryb === "latwy") ? "latwy" : "normalny";
    if(trybGry === "latwy") {
        maxGwiazdki = 7;
        gwiazdki = 7;
    } else {
        maxGwiazdki = 5;
        gwiazdki = 5;
    }
    document.getElementById("val-gwiazdki").innerText = "⭐".repeat(gwiazdki);
    document.getElementById("start-menu").style.display = "none";
    renderujTaskUI();
    nowyKlient();
}

function kupStyl(styl,koszt){
    if(!zakupioneStyle.includes(styl)){
       if(zarobek<koszt) { playSound(150, 0.2, 'sawtooth'); return; }
       zarobek-=koszt;
       zakupioneStyle.push(styl);
       document.getElementById("val-zarobek").innerText=zarobek.toFixed(2)+" zł";
       let btn= document.getElementById("btn-skin-"+styl);
       if(btn) btn.innerText="POSIADASZ";
    }

    aktualnyStyl = styl;
    PALETA=motywySklepu[styl];
    
    scene.background = new THREE.Color(PALETA.tlo);
    sciana.material.map = generujPixelTexture("standard", PALETA.tlo).map;
    sciana.material.needsUpdate = true;
    lada.material.map = generujPixelTexture("standard", PALETA.lada).map;
    lada.material.needsUpdate = true;
    window.matRegal.map = generujPixelTexture("standard", PALETA.regal).map;
    window.matRegal.needsUpdate = true;
    
    playSound(600, 0.15, 'sine');
}

// === RADIO KORPO - PROCEDURALNA MUZYKA 8-BIT ===
let radioPlaying = false;
let radioAudioCtx = null;
let radioTimeout = null;
let radioStep = 0;
const RADIO_STEP_TIME = 0.18; // ~133 BPM (16 kroków)

const RADIO_BASS = [
    130.81,130.81,0,130.81, 196.00,196.00,0,196.00,
    174.61,174.61,0,174.61, 196.00,196.00,0,0,
    130.81,130.81,0,130.81, 196.00,196.00,0,196.00,
    174.61,174.61,0,174.61, 220.00,220.00,0,0
];
const RADIO_LEAD = [
    523.25,0,587.33,659.25, 0,587.33,523.25,0,
    493.88,0,523.25,587.33, 659.25,783.99,659.25,587.33,
    523.25,0,587.33,659.25, 0,587.33,523.25,0,
    493.88,0,523.25,587.33, 659.25,783.99,0,0
];
const RADIO_DRUM = [
    1,1,1,1, 0,1,1,0, 1,0,1,0, 1,1,1,0,
    1,1,1,1, 0,1,1,0, 1,0,1,0, 1,1,1,0
];

function radioTone(freq, type, time, dur, vol) {
    if(!radioAudioCtx) return;
    let o = radioAudioCtx.createOscillator();
    let g = radioAudioCtx.createGain();
    o.type = type;
    o.frequency.setValueAtTime(freq, time);
    g.gain.setValueAtTime(0.0001, time);
    g.gain.exponentialRampToValueAtTime(vol, time + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, time + dur);
    o.connect(g); g.connect(radioAudioCtx.destination);
    o.start(time); o.stop(time + dur + 0.02);
}
function radioNoise(time, dur) {
    if(!radioAudioCtx) return;
    let bufSize = Math.max(1, Math.floor(radioAudioCtx.sampleRate * dur));
    let buf = radioAudioCtx.createBuffer(1, bufSize, radioAudioCtx.sampleRate);
    let data = buf.getChannelData(0);
    for(let i=0;i<bufSize;i++) data[i] = Math.random()*2-1;
    let n = radioAudioCtx.createBufferSource();
    n.buffer = buf;
    let g = radioAudioCtx.createGain();
    g.gain.setValueAtTime(0.08, time);
    g.gain.exponentialRampToValueAtTime(0.0001, time+dur);
    n.connect(g); g.connect(radioAudioCtx.destination);
    n.start(time); n.stop(time+dur);
}
function radioScheduleStep() {
    if(!radioPlaying || !radioAudioCtx) return;
    let t = radioAudioCtx.currentTime + 0.05;
    let bIdx = radioStep % RADIO_BASS.length;
    let lIdx = radioStep % RADIO_LEAD.length;
    let dIdx = radioStep % RADIO_DRUM.length;
    if(RADIO_BASS[bIdx]) radioTone(RADIO_BASS[bIdx], 'triangle', t, RADIO_STEP_TIME*1.8, 0.18);
    if(RADIO_LEAD[lIdx]) radioTone(RADIO_LEAD[lIdx], 'square', t, RADIO_STEP_TIME*0.9, 0.12);
    if(RADIO_DRUM[dIdx]) {
        radioTone(120, 'square', t, 0.06, 0.25);
        radioNoise(t, 0.05);
    }
    radioStep++;
    radioTimeout = setTimeout(radioScheduleStep, RADIO_STEP_TIME * 1000);
}
function toggleRadio() {
    let btn = document.getElementById('radio-btn');
    if(radioPlaying) {
        radioPlaying = false;
        if(radioTimeout) clearTimeout(radioTimeout);
        btn.classList.remove('playing');
        return;
    }
    try {
        if(!radioAudioCtx) radioAudioCtx = new (window.AudioContext || window.webkitAudioContext)();
        let start = () => {
            radioPlaying = true;
            radioStep = 0;
            btn.classList.add('playing');
            radioScheduleStep();
        };
        if(radioAudioCtx.state === 'suspended') {
            radioAudioCtx.resume().then(start).catch(start);
        } else {
            start();
        }
    } catch(e) {
        console.error('Radio error:', e);
        alert('Twoja przeglądarka nie wspiera Web Audio.');
    }
}
</script>

<div id="black-market">
    <h2>💻 CZARNY RYNEK</h2>
    <div class="bm-sub">// nielegalne burgery z tymczasowymi ulepszeniami //</div>
    <div class="burger-item" data-typ="moc">
        <div class="b-info">🍔 <b>Burger Mocy</b> — 200 zł<div class="b-desc">Taśma x2 prędkości przez 30s</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('moc')">KUP</button>
    </div>
    <div class="burger-item" data-typ="big">
        <div class="b-info">🍔 <b>Big Energy</b> — 150 zł<div class="b-desc">Pełna regeneracja energii kasjera</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('big')">KUP</button>
    </div>
    <div class="burger-item" data-typ="zen">
        <div class="b-info">🍔 <b>Burger Zen</b> — 100 zł<div class="b-desc">Cierpliwość obecnego klienta na max</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('zen')">KUP</button>
    </div>
    <div class="burger-item" data-typ="combo">
        <div class="b-info">🍔 <b>Combo Burger</b> — 250 zł<div class="b-desc">+3 do combo natychmiast</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('combo')">KUP</button>
    </div>
    <div class="burger-item" data-typ="zloty">
        <div class="b-info">🍔 <b>Złoty Burger</b> — 400 zł<div class="b-desc">Mnożnik zysku x2 przez 45s</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('zloty')">KUP</button>
    </div>
    <div class="burger-item" data-typ="haker">
        <div class="b-info">🍔 <b>Burger Hakera</b> — 500 zł<div class="b-desc">Blok awarii / BHP / złodziei na 60s</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupBurger('haker')">KUP</button>
    </div>
    <div class="burger-item" data-typ="opinia">
        <div class="b-info">⭐ <b>Lewa Opinia</b> — 1000 zł<div class="b-desc">Odzyskaj jedną gwiazdkę opinii</div><div class="b-lock"></div></div>
        <button class="burger-btn" onclick="kupOpinie()">KUP</button>
    </div>
    <div id="bm-close-hint">[D] — wróć do kasy</div>
</div>

<div id="hint-obrot">[A] laptop  |  [D] magazyn</div>

<button id="btn-jak-grac" onclick="togglePomoc()">❓ JAK GRAĆ</button>
<div id="panel-pomoc">
    <div class="ph-title">📖 JAK GRAĆ</div>
    <div class="ph-sek"><b>🛒 KASA</b><br>
        • Skanuj produkty kierując je na laser (czerwona linia)<br>
        • Wpisuj kwotę na klawiaturze, klient płaci automatycznie<br>
        • Pilnuj kolejki - zbyt długie czekanie = stracony klient
    </div>
    <div class="ph-sek"><b>💻 LAPTOP / CZARNY RYNEK</b><br>
        • Wciśnij <b>[A]</b> aby obrócić się w lewo do laptopa<br>
        • Kupuj burgery dające tymczasowe ulepszenia<br>
        • Nowe burgery odblokowują się co kilka dni (4, 8, 12, 16, 20, 24)
    </div>
    <div class="ph-sek"><b>📦 MAGAZYN (z tyłu)</b><br>
        • Wciśnij <b>[D]</b> aby spojrzeć do magazynu<br>
        • Co jakiś czas (7% szansy) wchodzi <span style="color:#ff004d">złodziej</span><br>
        • Gdy zobaczysz złodzieja w drzwiach masz <b>4 sekundy</b><br>
        • Wciśnij <b>[SPACJA]</b> żeby go przegonić<br>
        • Jeśli nie zdążysz - tracisz <b>300 zł</b> z salda<br>
        • <span style="color:#ffec27">Burger Hakera</span> blokuje złodziei na 60s
    </div>
    <div class="ph-sek"><b>⬆️ ULEPSZENIA</b><br>
        • Inwestuj w taśmę, kasę i prestiż między dniami<br>
        • Wyższy prestiż = więcej klientów i lepsze produkty
    </div>
    <div class="ph-close" onclick="togglePomoc()">[X] ZAMKNIJ</div>
</div>

<script>
    if(typeof odswiezBurgeryUI === 'function') odswiezBurgeryUI();
    if(typeof aktualizujHintObrot === 'function') aktualizujHintObrot();
    function togglePomoc(){
        let p = document.getElementById('panel-pomoc');
        p.style.display = (p.style.display === 'block') ? 'none' : 'block';
    }
</script>

<button id="radio-btn" onclick="toggleRadio()" title="Włącz / wyłącz radio"><span class="radio-led"></span>📻 RADIO KORPO</button>

</body>
</html>
