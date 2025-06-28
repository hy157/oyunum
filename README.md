<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <title>Tarım Oyunu - Zoom Güvenli</title>
  <style>
    body {
      background: #4ecb36;
      margin: 0;
      overflow: hidden;
      width: 100vw; height: 100vh;
    }
    #game-container {
      position: fixed;
      left: 0; top: 0;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      z-index: 1;
      touch-action: none;
      background: transparent;
    }
    #toolbar {
      position: absolute;
      top: 10px; left: 10px; z-index: 10;
      display: flex; align-items: center;
    }
    .tool-btn {
      font-size: 2rem;
      padding: 8px 16px;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      background: #fff9;
      margin-right: 10px;
      box-shadow: 0 2px 8px #0002;
      outline: 2px solid transparent;
      transition: outline 0.2s;
    }
    .tool-btn.active {
      outline: 2px solid #13bb18;
      background: #baffc9;
    }
    .status-bars { position: absolute; top: 10px; right: 30px; z-index: 20; display: flex; gap: 14px; }
    #money-bar, #wheat-bar { background: #fff8; padding: 10px 22px 10px 18px; border-radius: 20px; font-size: 2rem; box-shadow: 0 2px 8px #0002; display: flex; align-items: center; font-weight: bold; gap: 8px; min-width: 88px; user-select: none; height: 48px; }
    #money-bar { min-width: 88px; }
    #wheat-bar { min-width: 88px; }
    #money-amount, #wheat-amount { font-family: monospace; font-size: 2rem; transition: color 0.2s, transform 0.2s; }
    #money-bar.money-gain #money-amount { color: #09c517; transform: scale(1.15); }
    #money-bar.money-lose #money-amount { color: #d62424; transform: scale(0.9); }
    #wheat-price-bar {
      background: #ffe7aaee;
      padding: 10px 20px 10px 14px;
      border-radius: 20px;
      font-size: 2rem;
      box-shadow: 0 2px 8px #0002;
      display: flex;
      align-items: center;
      font-weight: bold;
      gap: 8px;
      min-width: 98px;
      user-select: none;
      height: 48px;
      cursor: pointer;
      transition: box-shadow 0.2s, background 0.2s;
      border: 2px solid #ffb60066;
      margin-left: 8px;
      margin-right: 8px;
    }
    #wheat-price-bar:active { box-shadow: 0 2px 16px #ffa60055; background: #ffefb3; }
    #wheat-price-icon { width: 48px; height: 48px; object-fit: contain; margin-right: 2px; user-select: none; pointer-events: none; }
    #wheat-price-amount { font-family: monospace; font-size: 2rem; color: #c38307; margin-right: 2px; min-width: 28px; display: inline-block; transition: color 0.2s, transform 0.2s; }
    #wheat-price-label { font-size: 1.3rem; color: #9a7e00; margin-left: 0; }
    #wheat-price-bar.price-gain #wheat-price-amount { color: #1a9701; transform: scale(1.18); }
    #wheat-price-bar.price-sell #wheat-price-amount { color: #ff1b1b; transform: scale(1.25); }
    #game-area {
      position: absolute;
      left: 0; top: 0;
      width: 1920px; height: 1080px;
      cursor: default;
      user-select: none;
      z-index: 2;
    }
    .iso-plot {
      position: absolute;
      width: 192px;
      height: 128px;
      transform: translate(-50%, -50%);
      pointer-events: none;
      transition: filter 0.1s;
    }
    .fall-anim { animation: fall-in 0.4s cubic-bezier(.55,1.5,.58,1) both; }
    @keyframes fall-in {
      from { opacity: 0; transform: translate(-50%, -130%) scaleY(1.2); filter: blur(2px);}
      80% { opacity: 1; transform: translate(-50%, -56%) scaleY(0.95); filter: blur(0);}
      to { opacity: 1; transform: translate(-50%, -50%) scaleY(1); filter: blur(0);}
    }
    .fly-anim { animation: fly-out 0.4s cubic-bezier(.7,-0.4,1,.57) both; z-index: 10; }
    @keyframes fly-out {
      from { opacity: 1; transform: translate(-50%, -50%) scaleY(1); filter: blur(0);}
      40% { opacity: 1; filter: blur(0.5px); transform: translate(-50%, -70%) scaleY(1.05);}
      to { opacity: 0; filter: blur(2px); transform: translate(-50%, -130%) scaleY(1.2);}
    }
    .money-float {
      position: absolute; left: 0; top: 0; pointer-events: none;
      font-size: 1.7rem; font-weight: bold; opacity: 1; z-index: 99;
      transition: opacity 0.15s; will-change: transform, opacity; user-select: none;
    }
    .money-gain { color: #11c90e; text-shadow: 0 1px 4px #fff8,0 0 2px #45ff42; }
    .money-lose { color: #d62424; text-shadow: 0 1px 4px #fff8,0 0 2px #ff6961; }
    #wheat-icon { width: 64px; height: 64px; object-fit: contain; margin-right: 6px; vertical-align: middle; user-select: none; pointer-events: none; }
  </style>
</head>
<body>
  <audio id="sound-coin" src="https://cdn.pixabay.com/audio/2022/07/26/audio_124bfa4f88.mp3"></audio>
  <audio id="sound-pay" src="https://cdn.pixabay.com/audio/2022/03/15/audio_115bfae13d.mp3"></audio>

  <div id="game-container">
    <div id="game-area"></div>
  </div>
  <div id="toolbar">
    <button id="pickaxe-btn" class="tool-btn">⛏️ Kazma <kbd>1</kbd></button>
    <button id="wheat-btn" class="tool-btn">🌾 Buğday <kbd>2</kbd></button>
    <button id="sickle-btn" class="tool-btn">🔪 Orak <kbd>3</kbd></button>
    <button id="remove-btn" class="tool-btn">❌ Çarpı <kbd>4</kbd></button>
  </div>
  <div class="status-bars">
    <div id="money-bar">🪙 <span id="money-amount">100</span></div>
    <div id="wheat-price-bar" title="Elindeki fazla buğdayı sat">
      <img id="wheat-price-icon" src="img/wheat.webp" alt="Buğday">
      <span id="wheat-price-amount">10</span>
      <span id="wheat-price-label"></span>
    </div>
    <div id="wheat-bar">
      <img id="wheat-icon" src="img/wheat.webp" alt="Buğday">
      <span id="wheat-amount">10</span>
    </div>
  </div>

<script>
const VIRTUAL_W = 1920, VIRTUAL_H = 1080;
let scale = 1, offsetLeft = 0, offsetTop = 0;

const tileW = 192, tileH = 128;
const plotImgSrc = "img/Plot.webp";
const plot1ImgSrc = "img/Plot1.webp";
const plot2ImgSrc = "img/Plot2.webp";
const plot3ImgSrc = "img/Plot3.webp";
const wheatImgSrc = "img/wheat.webp";
const pickaxeCursor = "https://em-content.zobj.net/source/apple/354/pick_axe_26cf.png";
const removeCursor = "https://em-content.zobj.net/source/microsoft-teams/363/cross-mark_274c.png";

const soundCoin = document.getElementById('sound-coin');
const soundPay = document.getElementById('sound-pay');

let money = 100;
let wheat = 10;
const EKIM_MALIYET = 50;
const SILME_KAZANCI = 20;

let wheatPrice = 10;
const MIN_WHEAT_PRICE = 10;
const MAX_WHEAT_PRICE = 20;

const wheatPriceBar = document.getElementById('wheat-price-bar');
const wheatPriceAmount = document.getElementById('wheat-price-amount');

const pickaxeBtn = document.getElementById('pickaxe-btn');
const wheatBtn = document.getElementById('wheat-btn');
const sickleBtn = document.getElementById('sickle-btn');
const removeBtn = document.getElementById('remove-btn');
const gameArea = document.getElementById('game-area');
const moneyBar = document.getElementById('money-bar');
const moneyAmount = document.getElementById('money-amount');
const wheatBar = document.getElementById('wheat-bar');
const wheatAmount = document.getElementById('wheat-amount');
let isPickaxe = false;
let isRemove = false;
let isWheat = false;
let isSickle = false;
let isMouseDown = false;
let plots = [];
let wheats = [];
let handledCoords = new Set();

function updateGameAreaTransform() {
  let container = document.getElementById('game-container');
  let ww = window.innerWidth, wh = window.innerHeight;
  scale = Math.min(ww / VIRTUAL_W, wh / VIRTUAL_H);
  offsetLeft = (ww - VIRTUAL_W * scale) / 2;
  offsetTop = (wh - VIRTUAL_H * scale) / 2;
  gameArea.style.transform = `scale(${scale})`;
  gameArea.style.left = `${offsetLeft / scale}px`;
  gameArea.style.top = `${offsetTop / scale}px`;
  gameArea.style.transformOrigin = '0 0';
}
updateGameAreaTransform();
window.addEventListener('resize', () => {
  updateGameAreaTransform();
  redrawAllPlots();
});

function screenToGame(x, y) {
  return {
    x: (x - offsetLeft) / scale,
    y: (y - offsetTop) / scale
  };
}
function gameToScreen(x, y) {
  return {
    x: x * scale + offsetLeft,
    y: y * scale + offsetTop
  };
}
function isoToScreen(x, y) {
  const centerX = VIRTUAL_W / 2;
  const centerY = VIRTUAL_H / 2;
  return {
    left: centerX + (x - y) * (tileW / 2),
    top: centerY + (x + y) * (tileH / 2)
  };
}
function screenToIso(screenX, screenY) {
  const {x, y} = screenToGame(screenX, screenY);
  const centerX = VIRTUAL_W / 2;
  const centerY = VIRTUAL_H / 2;
  const mouseX = x - centerX;
  const mouseY = y - centerY;
  let ix = Math.round((mouseY / (tileH / 2) + mouseX / (tileW / 2)) / 2);
  let iy = Math.round((mouseY / (tileH / 2) - mouseX / (tileW / 2)) / 2);
  return {x: ix, y: iy};
}

function updateMoneyBar(type) {
  moneyAmount.textContent = money;
  moneyBar.classList.remove('money-gain', 'money-lose');
  void moneyBar.offsetWidth;
  if (type === 'lose') moneyBar.classList.add('money-lose');
  else if (type === 'gain') moneyBar.classList.add('money-gain');
  setTimeout(() => moneyBar.classList.remove('money-gain', 'money-lose'), 350);
}
function updateWheatBar() { wheatAmount.textContent = wheat; }
function updateWheatPriceBar(animType) {
  wheatPriceAmount.textContent = wheatPrice;
  wheatPriceBar.classList.remove('price-gain', 'price-sell');
  void wheatPriceBar.offsetWidth;
  if(animType === 'gain') { wheatPriceBar.classList.add('price-gain'); setTimeout(()=>wheatPriceBar.classList.remove('price-gain'), 350);}
  if(animType === 'sell') { wheatPriceBar.classList.add('price-sell'); setTimeout(()=>wheatPriceBar.classList.remove('price-sell'), 450);}
}
setInterval(() => {
  wheatPrice = Math.floor(Math.random() * (MAX_WHEAT_PRICE - MIN_WHEAT_PRICE + 1)) + MIN_WHEAT_PRICE;
  updateWheatPriceBar('gain');
}, 1000);

wheatPriceBar.addEventListener('click', () => {
  const satilacak = Math.max(0, wheat - 10);
  if (satilacak === 0) return;
  const kazanc = satilacak * wheatPrice;
  const wheatBarRect = wheatBar.getBoundingClientRect();
  const priceBarRect = wheatPriceBar.getBoundingClientRect();
  animateWheatFly({
    from: { x: wheatBarRect.left + wheatBarRect.width/2 - 32, y: wheatBarRect.top + wheatBarRect.height/2 - 32 },
    to:   { x: priceBarRect.left + priceBarRect.width/2 - 32, y: priceBarRect.top + priceBarRect.height/2 - 32 },
    count: Math.min(satilacak, 7)
  });
  setTimeout(() => {
    const moneyBarRect = moneyBar.getBoundingClientRect();
    animateMoneyFloat({
      amount: kazanc,
      from: { x: priceBarRect.left + priceBarRect.width/2 - 12, y: priceBarRect.top + priceBarRect.height/2 - 12 },
      to:   { x: moneyBarRect.left + moneyBarRect.width/2 - 12, y: moneyBarRect.top + moneyBarRect.height/2 - 12 }
    });
    soundCoin.currentTime = 0; soundCoin.play();
  }, 300);
  wheat -= satilacak;
  updateWheatBar();
  setTimeout(() => {
    money += kazanc;
    updateMoneyBar('gain');
    updateWheatPriceBar('sell');
  }, 450);
});
function animateMoneyFloat({amount, from, to}) {
  const emoji = '🪙';
  const sign = amount > 0 ? '+' : '';
  const colorClass = amount > 0 ? 'money-gain' : 'money-lose';
  const float = document.createElement('span');
  float.className = `money-float ${colorClass}`;
  float.textContent = `${sign}${amount} ${emoji}`;
  float.style.left = from.x + 'px';
  float.style.top = from.y + 'px';
  float.style.opacity = 1;
  document.body.appendChild(float);
  float.animate([
    {transform: 'translate(0,0) scale(1)',opacity:1},
    {transform: `translate(${to.x-from.x}px,${to.y-from.y}px) scale(1.4)`,opacity:1,offset:0.8},
    {transform: `translate(${to.x-from.x}px,${to.y-from.y}px) scale(0.7)`,opacity:0}
  ], {duration: 700,easing:'cubic-bezier(.57,.11,.82,1.39)'});
  setTimeout(() => { float.remove(); }, 750);
}
function animateWheatFly({from, to, count=1}) {
  for (let i = 0; i < count; i++) {
    const icon = document.createElement('img');
    icon.src = wheatImgSrc;
    icon.style.position = 'absolute';
    icon.style.left = from.x + 'px';
    icon.style.top = from.y + 'px';
    icon.style.width = '96px';
    icon.style.height = '96px';
    icon.style.pointerEvents = 'none';
    icon.style.opacity = 1;
    icon.style.zIndex = 100;
    icon.style.transition = 'none';
    document.body.appendChild(icon);
    const startOffsetX = (Math.random()-0.5)*16;
    const startOffsetY = (Math.random()-0.5)*16;
    icon.style.transform = `translate(${startOffsetX}px,${startOffsetY}px) scale(1)`;
    setTimeout(() => {
      icon.animate([
        { transform: `translate(${startOffsetX}px,${startOffsetY}px) scale(1)`, opacity: 1 },
        { transform: `translate(${to.x-from.x}px,${to.y-from.y}px) scale(1.4)`, opacity: 1, offset: 0.7 },
        { transform: `translate(${to.x-from.x}px,${to.y-from.y}px) scale(0.7)`, opacity: 0 }
      ], { duration: 850 + i*80, easing: 'cubic-bezier(.45,1.2,.49,1)' });
      setTimeout(() => icon.remove(), 900 + i*80);
    }, i * 100);
  }
}

function setToolMode(mode) {
  if (mode === "pickaxe") {
    isPickaxe = !isPickaxe;
    if (isPickaxe) { isRemove = false; isWheat = false; isSickle = false; }
  } else if (mode === "remove") {
    isRemove = !isRemove;
    if (isRemove) { isPickaxe = false; isWheat = false; isSickle = false; }
  } else if (mode === "wheat") {
    isWheat = !isWheat;
    if (isWheat) { isPickaxe = false; isRemove = false; isSickle = false; }
  } else if (mode === "sickle") {
    isSickle = !isSickle;
    if (isSickle) { isPickaxe = false; isRemove = false; isWheat = false; }
  }
  pickaxeBtn.classList.toggle('active', isPickaxe);
  removeBtn.classList.toggle('active', isRemove);
  wheatBtn.classList.toggle('active', isWheat);
  sickleBtn.classList.toggle('active', isSickle);

  if (isPickaxe) gameArea.style.cursor = `url(${pickaxeCursor}) 16 16, pointer`;
  else if (isRemove) gameArea.style.cursor = `url(${removeCursor}) 16 16, pointer`;
  else if (isWheat || isSickle) gameArea.style.cursor = "pointer";
  else gameArea.style.cursor = "default";
}

pickaxeBtn.addEventListener('click', () => setToolMode("pickaxe"));
removeBtn.addEventListener('click', () => setToolMode("remove"));
wheatBtn.addEventListener('click', () => setToolMode("wheat"));
sickleBtn.addEventListener('click', () => setToolMode("sickle"));

document.body.addEventListener('keydown', (e) => {
  if (e.key === "1") setToolMode("pickaxe");
  if (e.key === "2") setToolMode("wheat");
  if (e.key === "3") setToolMode("sickle");
  if (e.key === "4") setToolMode("remove");
  if (e.key === "Escape") {
    isPickaxe = false; isRemove = false; isWheat = false; isSickle = false;
    pickaxeBtn.classList.remove('active'); removeBtn.classList.remove('active');
    wheatBtn.classList.remove('active'); sickleBtn.classList.remove('active');
    gameArea.style.cursor = "default";
  }
});

function plotId(x, y) { return `plot-${x}-${y}`; }

function addPlot(x, y, mousePos = null) {
  if (plots.find(p => p.x === x && p.y === y)) return false;
  if (money < EKIM_MALIYET) return false;
  plots.push({x, y});
  const coords = isoToScreen(x, y);
  const img = document.createElement('img');
  img.src = plotImgSrc;
  img.className = "iso-plot";
  img.style.left = coords.left + "px";
  img.style.top = coords.top + "px";
  img.id = plotId(x, y);
  if (isPickaxe) img.classList.add('fall-anim');
  gameArea.appendChild(img);
  money -= EKIM_MALIYET;
  updateMoneyBar('lose');
  if (mousePos) {
    const barRect = moneyBar.getBoundingClientRect();
    const from = { x: barRect.left + barRect.width / 2 - 10, y: barRect.top + barRect.height / 2 - 14 };
    animateMoneyFloat({ amount: -EKIM_MALIYET, from: from, to: mousePos });
    soundPay.currentTime = 0; soundPay.play();
  }
  return true;
}

// Ekin ekildiğinde timestamp ile kaydet!
function addWheat(x, y) {
  if (!plots.find(p => p.x === x && p.y === y)) return false;
  if (wheats.find(w => w.x === x && w.y === y)) return false;
  if (wheat < 1) return false;
  const barRect = wheatBar.getBoundingClientRect();
  const coords = isoToScreen(x, y);
  const plotPos = gameToScreen(coords.left, coords.top);
  const barPos = { x: barRect.left + barRect.width / 2 - 16, y: barRect.top + barRect.height / 2 - 16 };
  animateWheatFly({ from: barPos, to: plotPos, count: 1 });
  wheat--;
  updateWheatBar();
  const plotImg = document.getElementById(plotId(x, y));
  if (!plotImg) return false;

  // Ekin ekilme zamanı
  const now = Date.now();
  let timerDiv = document.createElement('div');
  timerDiv.style.position = "absolute";
  timerDiv.style.left = (coords.left - 42) + "px";
  timerDiv.style.top = (coords.top - 58) + "px";
  timerDiv.style.width = "84px";
  timerDiv.style.height = "34px";
  timerDiv.style.background = "rgba(255,255,255,0.85)";
  timerDiv.style.borderRadius = "10px";
  timerDiv.style.textAlign = "center";
  timerDiv.style.fontWeight = "bold";
  timerDiv.style.fontSize = "1.3rem";
  timerDiv.style.lineHeight = "34px";
  timerDiv.style.pointerEvents = "none";
  timerDiv.id = `timer-${x}-${y}`;

  // Her ekin için bilgileri timestamp ile tutuyoruz
  let wheatObj = {
    x, y,
    plantedAt: now,
    ready: false,
    timerEl: timerDiv,
    timerInterval: null,
    harvested: false
  };

  function updateTimer() {
    const elapsed = Math.floor((Date.now() - wheatObj.plantedAt) / 1000);
    const remain = Math.max(0, 6 - elapsed);

    if (wheatObj.harvested) return;

    // AŞAMA: 6-4-3-2-1-0
    if (remain > 3) {
      plotImg.src = plot1ImgSrc;
    } else if (remain > 0) {
      plotImg.src = plot2ImgSrc;
    } else {
      plotImg.src = plot3ImgSrc;
    }

    if (remain > 0) {
      timerDiv.textContent = remain;
      timerDiv.style.background = "rgba(255,255,255,0.85)";
      timerDiv.style.color = "#000";
    } else {
      timerDiv.textContent = "✔";
      timerDiv.style.background = "#baffc9";
      timerDiv.style.color = "#09980a";
      wheatObj.ready = true;
      clearInterval(wheatObj.timerInterval);
    }
  }

  updateTimer();
  wheatObj.timerInterval = setInterval(updateTimer, 400);

  gameArea.appendChild(timerDiv);
  wheats.push(wheatObj);
  return true;
}

function harvestWheat(x, y) {
  const wheatObj = wheats.find(w => w.x === x && w.y === y);
  if (!wheatObj || !wheatObj.ready || wheatObj.harvested) return false;
  wheatObj.harvested = true;
  const plotImg = document.getElementById(plotId(x, y));
  if (plotImg) plotImg.src = plotImgSrc;
  const plotRect = plotImg.getBoundingClientRect();
  const barRect = wheatBar.getBoundingClientRect();
  const from = { x: plotRect.left + plotRect.width/2 - 48, y: plotRect.top + plotRect.height/2 - 48 };
  const to = { x: barRect.left + barRect.width/2 - 48, y: barRect.top + barRect.height/2 - 48 };
  animateWheatFly({from, to, count: 4});
  if (wheatObj.timerEl) wheatObj.timerEl.remove();
  if (wheatObj.timerInterval) clearInterval(wheatObj.timerInterval);
  wheats = wheats.filter(w => !(w.x === x && w.y === y));
  setTimeout(() => { wheat += 4; updateWheatBar(); }, 500);
  return true;
}

function removePlot(x, y, mousePos = null) {
  const idx = plots.findIndex(p => p.x === x && p.y === y);
  if (idx !== -1) {
    plots.splice(idx, 1);
    const el = document.getElementById(plotId(x, y));
    if (el) {
      el.classList.add('fly-anim');
      el.addEventListener('animationend', () => { if (el && el.parentNode) el.parentNode.removeChild(el); }, {once:true});
    }
    const wheatIdx = wheats.findIndex(w => w.x === x && w.y === y);
    if (wheatIdx !== -1) {
      const wheatObj = wheats[wheatIdx];
      if (wheatObj.timerEl) wheatObj.timerEl.remove();
      if (wheatObj.timerInterval) clearInterval(wheatObj.timerInterval);
      wheats.splice(wheatIdx, 1);
    }
    money += SILME_KAZANCI;
    updateMoneyBar('gain');
    if (mousePos) {
      const barRect = moneyBar.getBoundingClientRect();
      const to = { x: barRect.left + barRect.width / 2 - 10, y: barRect.top + barRect.height / 2 - 14 };
      animateMoneyFloat({ amount: SILME_KAZANCI, from: mousePos, to: to });
      soundCoin.currentTime = 0; soundCoin.play();
    }
    return true;
  }
  return false;
}

function handleActionAtMouse(e) {
  let clientX, clientY;
  if (e.touches && e.touches.length > 0) {
    clientX = e.touches[0].clientX;
    clientY = e.touches[0].clientY;
  } else {
    clientX = e.clientX; clientY = e.clientY;
  }
  const { x, y } = screenToIso(clientX, clientY);
  const key = `${x},${y}`;
  if (handledCoords.has(key)) return;
  handledCoords.add(key);
  const mousePos = {x: clientX, y: clientY};
  if (isPickaxe) addPlot(x, y, mousePos);
  if (isRemove) removePlot(x, y, mousePos);
  if (isWheat) addWheat(x, y);
  if (isSickle) harvestWheat(x, y);
}

gameArea.addEventListener('mousedown', (e) => {
  if (isPickaxe || isRemove || isWheat || isSickle) {
    isMouseDown = true;
    handledCoords.clear();
    handleActionAtMouse(e);
  }
});
gameArea.addEventListener('mousemove', (e) => {
  if (isMouseDown && (isPickaxe || isRemove || isWheat || isSickle)) {
    handleActionAtMouse(e);
  }
});
document.addEventListener('mouseup', () => { isMouseDown = false; handledCoords.clear(); });
gameArea.addEventListener('mouseleave', () => { isMouseDown = false; handledCoords.clear(); });
gameArea.addEventListener('touchstart', (e) => { if (isPickaxe || isRemove || isWheat || isSickle) { isMouseDown = true; handledCoords.clear(); handleActionAtMouse(e); } });
gameArea.addEventListener('touchmove', (e) => { if (isMouseDown && (isPickaxe || isRemove || isWheat || isSickle)) { handleActionAtMouse(e); } });
gameArea.addEventListener('touchend', () => { isMouseDown = false; handledCoords.clear(); });

function redrawAllPlots() {
  document.querySelectorAll('.iso-plot').forEach(el => el.remove());
  document.querySelectorAll('[id^="timer-"]').forEach(el => el.remove());
  // Tarlalar
  plots.forEach(({x, y}) => {
    const coords = isoToScreen(x, y);
    const img = document.createElement('img');
    img.src = plotImgSrc;
    img.className = "iso-plot";
    img.style.left = coords.left + "px";
    img.style.top = coords.top + "px";
    img.id = plotId(x, y);
    gameArea.appendChild(img);
  });
  // Ekinler ve timerlar
  wheats.forEach(w => {
    const coords = isoToScreen(w.x, w.y);
    const plotImg = document.getElementById(plotId(w.x, w.y));
    if (!plotImg) return;
    // Kalan süre
    const elapsed = Math.floor((Date.now() - w.plantedAt) / 1000);
    const remain = Math.max(0, 6 - elapsed);

    if (w.harvested) return;

    if (remain > 3) {
      plotImg.src = plot1ImgSrc;
    } else if (remain > 0) {
      plotImg.src = plot2ImgSrc;
    } else {
      plotImg.src = plot3ImgSrc;
    }

    // Timerı güncelle
    let timerDiv = document.createElement('div');
    timerDiv.style.position = "absolute";
    timerDiv.style.left = (coords.left - 42) + "px";
    timerDiv.style.top = (coords.top - 58) + "px";
    timerDiv.style.width = "84px";
    timerDiv.style.height = "34px";
    timerDiv.style.borderRadius = "10px";
    timerDiv.style.textAlign = "center";
    timerDiv.style.fontWeight = "bold";
    timerDiv.style.fontSize = "1.3rem";
    timerDiv.style.lineHeight = "34px";
    timerDiv.style.pointerEvents = "none";
    timerDiv.id = `timer-${w.x}-${w.y}`;
    if (remain > 0) {
      timerDiv.textContent = remain;
      timerDiv.style.background = "rgba(255,255,255,0.85)";
      timerDiv.style.color = "#000";
    } else {
      timerDiv.textContent = "✔";
      timerDiv.style.background = "#baffc9";
      timerDiv.style.color = "#09980a";
      w.ready = true;
    }
    gameArea.appendChild(timerDiv);
    w.timerEl = timerDiv;
  });
}

document.getElementById('wheat-icon').src = wheatImgSrc;
document.getElementById('wheat-price-icon').src = wheatImgSrc;
updateWheatBar();
updateWheatPriceBar();

addPlot(0, 0);
addPlot(1, 0);
addPlot(0, 1);

</script>
</body>
</html>
