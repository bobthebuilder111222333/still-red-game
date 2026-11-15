doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>STILL / RED</title>
<style>
  :root { color-scheme: dark; font-family: "Segoe UI", Arial, sans-serif; }
  * { box-sizing: border-box; }
  [hidden] { display: none !important; }
  html, body { width: 100%; height: 100%; margin: 0; overflow: hidden; background: #101113; }
  canvas { display: block; width: 100%; height: 100%; }
  #hud { position: fixed; inset: 0; z-index: 3; pointer-events: none; color: #f0eee9; }
  body[data-game-state="menu"] #hud { visibility: hidden; }
  #hud:before { content: ""; position: absolute; inset: 0; background: radial-gradient(ellipse at center, transparent 53%, rgba(3,6,12,.24) 100%); }
  #hud:after { content: ""; position: absolute; inset: 16px; border: 1px solid rgba(220,230,240,.12); clip-path: polygon(0 0,48px 0,48px 1px,1px 1px,1px 48px,0 48px,0 calc(100% - 48px),1px calc(100% - 48px),1px calc(100% - 1px),48px calc(100% - 1px),48px 100%,0 100%,0 0,100% 0,calc(100% - 48px) 0,calc(100% - 48px) 1px,calc(100% - 1px) 1px,calc(100% - 1px) 48px,100% 48px,100% calc(100% - 48px),calc(100% - 1px) calc(100% - 48px),calc(100% - 1px) calc(100% - 1px),calc(100% - 48px) calc(100% - 1px),calc(100% - 48px) 100%,100% 100%); }
  .topline { position: absolute; top: 30px; left: 38px; right: 38px; display: flex; justify-content: space-between; align-items: flex-start; gap: 20px; }
  .brand { color: #f5f2e9; font-size: 13px; font-weight: 800; letter-spacing: .3em; text-shadow: 0 2px 8px #000; }
  .brand span { color: #f34d4d; }
  .mission { margin-top: 11px; color: rgba(235,239,243,.58); font-size: 9px; letter-spacing: .2em; text-shadow: 0 1px 5px #000; }
  .mission i { display: inline-block; width: 6px; height: 6px; margin-right: 7px; background: #f34d4d; box-shadow: 0 0 10px #f34d4d; }
  .stats { display: flex; gap: 0; text-align: right; border: 1px solid rgba(230,235,240,.16); background: linear-gradient(135deg,rgba(8,12,19,.78),rgba(8,12,19,.35)); box-shadow: 0 8px 28px #0002; backdrop-filter: blur(7px); }
  .stat { min-width: 76px; padding: 10px 14px 9px; border-left: 1px solid rgba(230,235,240,.12); }
  .stat:first-child { border-left: 0; }
  .stat-label { display: block; color: rgba(204,214,224,.55); font-size: 8px; letter-spacing: .2em; margin-bottom: 5px; }
  .stat-value { font-size: 17px; font-weight: 700; font-variant-numeric: tabular-nums; text-shadow: 0 1px 8px #000; }
  #replay-title { display: none; position: absolute; top: 102px; left: 38px; color: #f0eee9; font-size: 10px; font-weight: 800; letter-spacing: .24em; text-shadow: 0 2px 8px #000; }
  #freecam-label { display: none; position: absolute; top: 106px; left: 50%; transform: translateX(-50%); padding: 9px 13px; border: 1px solid rgba(121,231,220,.45); background: rgba(8,12,19,.72); color: #a7eee7; font-size: 9px; font-weight: 800; letter-spacing: .16em; white-space: nowrap; text-shadow: 0 1px 5px #000; }
  body[data-camera-mode="free"] #freecam-label { display: block; }
  #crosshair { position: absolute; left: 50%; top: 50%; width: 24px; height: 24px; transform: translate(-50%,-50%); opacity: .82; filter: drop-shadow(0 1px 3px #000); }
  #crosshair:before, #crosshair:after { content: ""; position: absolute; background: #e8eee8; }
  #crosshair:before { width: 1px; height: 24px; left: 11px; top: 0; }
  #crosshair:after { height: 1px; width: 24px; top: 11px; left: 0; }
  #crosshair i { position: absolute; width: 3px; height: 3px; left: 10px; top: 10px; border: 1px solid #f34d4d; border-radius: 50%; }
  body[data-camera-mode="free"] #crosshair { display: none; }
  #objective { position: absolute; left: 38px; bottom: 62px; width: 190px; padding: 11px 13px; border-left: 2px solid #f34d4d; background: linear-gradient(90deg,rgba(8,12,19,.68),transparent); text-shadow: 0 1px 5px #000; }
  #objective-label { color: rgba(204,214,224,.56); font-size: 8px; letter-spacing: .2em; }
  #objective-copy { margin-top: 5px; font-size: 11px; font-weight: 700; letter-spacing: .12em; }
  #objective-meter { height: 2px; margin-top: 9px; background: rgba(230,235,240,.18); }
  #objective-fill { width: 0; height: 100%; background: #f34d4d; transition: width .25s ease; box-shadow: 0 0 8px #f34d4d99; }
  #motion-status { display: none; position: absolute; right: 38px; bottom: 62px; align-items: center; gap: 10px; min-width: 170px; padding: 10px 12px; border: 1px solid rgba(230,235,240,.14); background: rgba(8,12,19,.55); backdrop-filter: blur(5px); }
  body[data-game-state="playing"] #motion-status, body[data-game-state="replay"] #motion-status { display: flex; }
  #motion-indicator { width: 7px; height: 7px; background: #f34d4d; box-shadow: 0 0 10px #f34d4d; }
  #motion-copy { color: rgba(236,240,244,.72); font-size: 8px; font-weight: 700; letter-spacing: .16em; }
  #motion-detail { margin-top: 4px; color: rgba(236,240,244,.42); font-size: 8px; letter-spacing: .12em; }
  body[data-motion="slow"] #motion-indicator { background: #79e7dc; box-shadow: 0 0 12px #79e7dc; animation: pulse 1.4s ease-in-out infinite; }
  body[data-motion="slow"] #motion-copy { color: #a7eee7; }
  @keyframes pulse { 50% { opacity: .38; } }
  #hint { position: absolute; bottom: 21px; left: 50%; transform: translateX(-50%); padding: 8px 13px; border: 1px solid rgba(230,235,240,.1); background: rgba(8,12,19,.48); color: rgba(220,225,230,.65); font-size: 9px; letter-spacing: .08em; white-space: nowrap; text-shadow: 0 1px 5px #000; }
  #toast { position: absolute; top: 23%; left: 50%; width: auto; max-width: calc(100vw - 40px); transform: translateX(-50%); padding: 9px 16px; border: 1px solid rgba(243,77,77,.55); background: rgba(14,11,14,.78); color: #ff8178; font-size: 10px; font-weight: 800; letter-spacing: .24em; opacity: 0; transition: opacity .15s; text-shadow: 0 0 12px #f34d4d66; }
  #toast.on { opacity: 1; }
  #overlay { position: fixed; inset: 0; display: grid; place-items: center; background: rgba(8,9,10,.78); color: #f1eee9; z-index: 4; }
  #panel { width: min(440px, calc(100vw - 40px)); padding: 38px; border: 1px solid #494644; background: rgba(19,20,21,.94); box-shadow: 0 24px 90px #0009; }
  #cheat-menu { position: fixed; inset: 0; z-index: 6; display: grid; place-items: center; padding: 20px; background: rgba(3,5,9,.88); color: #f1eee9; backdrop-filter: blur(8px); }
  #cheat-panel { width: min(430px,100%); padding: 30px; border: 1px solid rgba(243,77,77,.46); background: linear-gradient(145deg,rgba(18,20,24,.98),rgba(9,11,15,.98)); box-shadow: 0 24px 90px #000b,0 0 45px #f34d4d12; }
  #cheat-panel h2 { margin: 8px 0 10px; font-size: 30px; letter-spacing: -.04em; }
  #cheat-panel p { color: rgba(223,228,233,.68); font-size: 12px; line-height: 1.65; }
  #cheat-code-form { display: flex; gap: 8px; margin-top: 20px; }
  #cheat-code { min-width: 0; flex: 1; padding: 12px; border: 1px solid rgba(230,235,240,.24); outline: none; background: rgba(4,7,11,.8); color: #f1eee9; font: 700 15px "Segoe UI",Arial,sans-serif; letter-spacing: .2em; }
  #cheat-code:focus { border-color: #f34d4d; box-shadow: 0 0 0 2px #f34d4d22; }
  #cheat-code-form button, #cheat-close { padding: 11px 14px; border: 1px solid #a93c3c; background: #782929; color: #fff; font-size: 10px; font-weight: 800; letter-spacing: .12em; cursor: pointer; }
  #cheat-message { min-height: 18px; margin: 10px 0 4px; color: #f17b73 !important; font-size: 10px !important; font-weight: 700; letter-spacing: .12em; }
  #cheat-options { display: grid; gap: 8px; margin: 16px 0; }
  #cheat-options label { display: flex; align-items: center; gap: 11px; padding: 11px; border: 1px solid rgba(230,235,240,.12); background: rgba(255,255,255,.025); color: rgba(239,240,238,.85); font-size: 11px; font-weight: 700; letter-spacing: .08em; cursor: pointer; }
  #cheat-options input { width: 16px; height: 16px; accent-color: #f34d4d; }
  #cheat-close { width: 100%; margin-top: 8px; border-color: rgba(230,235,240,.2); background: rgba(255,255,255,.06); }
  #settings-menu { position: fixed; inset: 0; z-index: 8; display: grid; place-items: center; padding: 20px; background: rgba(3,5,9,.88); color: #f1eee9; backdrop-filter: blur(8px); }
  #settings-panel { width: min(430px,100%); padding: 30px; border: 1px solid rgba(243,77,77,.46); background: linear-gradient(145deg,rgba(18,20,24,.98),rgba(9,11,15,.98)); box-shadow: 0 24px 90px #000b; }
  #settings-panel h2 { margin: 8px 0 18px; font-size: 30px; letter-spacing: -.04em; }
  .setting-row { display: grid; gap: 10px; padding: 14px 0; border-top: 1px solid rgba(230,235,240,.12); }
  .setting-label { display: flex; justify-content: space-between; gap: 14px; color: rgba(239,240,238,.88); font-size: 11px; font-weight: 700; letter-spacing: .1em; }
  .setting-label output { color: #f17b73; font-variant-numeric: tabular-nums; }
  #sensitivity-setting { width: 100%; accent-color: #f34d4d; }
  .setting-note { color: rgba(223,228,233,.55); font-size: 10px; line-height: 1.5; }
  #palette-toggle { width: 16px; height: 16px; accent-color: #f34d4d; }
  #settings-close { width: 100%; margin-top: 16px; padding: 11px 14px; border: 1px solid #a93c3c; background: #782929; color: #fff; font-size: 10px; font-weight: 800; letter-spacing: .12em; cursor: pointer; }
  #level-select { position: fixed; inset: 0; z-index: 7; display: grid; place-items: center; padding: 20px; background: rgba(3,5,9,.88); color: #f1eee9; backdrop-filter: blur(8px); }
  #level-select-panel { width: min(520px,100%); padding: 30px; border: 1px solid rgba(243,77,77,.4); background: linear-gradient(145deg,rgba(18,20,24,.98),rgba(9,11,15,.98)); box-shadow: 0 24px 90px #000b; }
  #level-select-panel h2 { margin: 8px 0; font-size: 30px; letter-spacing: -.04em; }
  #level-progress { margin: 0 0 18px; color: rgba(223,228,233,.62); font-size: 11px; letter-spacing: .12em; }
  #level-grid { display: grid; grid-template-columns: repeat(2,minmax(0,1fr)); gap: 9px; }
  .level-button { display: flex; justify-content: space-between; align-items: center; gap: 10px; min-height: 54px; padding: 12px; border: 1px solid rgba(230,235,240,.2); background: rgba(255,255,255,.035); color: #f1eee9; font-size: 11px; font-weight: 800; letter-spacing: .1em; text-align: left; cursor: pointer; }
  .level-button:not(:disabled):hover { border-color: #f34d4d; background: rgba(243,77,77,.12); }
  .level-button:disabled { color: rgba(230,235,240,.35); cursor: not-allowed; }
  .level-button span:last-child { color: #f34d4d; font-size: 9px; }
  #level-select-close { width: 100%; margin-top: 15px; padding: 11px; border: 1px solid rgba(230,235,240,.2); background: rgba(255,255,255,.06); color: #fff; font-size: 10px; font-weight: 800; letter-spacing: .12em; cursor: pointer; }
  @media (max-width: 420px) { #level-select-panel { padding: 22px; } #level-grid { grid-template-columns: 1fr; gap: 7px; } .level-button { min-height: 45px; } }
  #overlay.start-screen { overflow: auto; display: block; padding: 0; background: linear-gradient(90deg,rgba(6,8,12,.84),rgba(7,9,13,.44) 54%,rgba(7,9,13,.22)); }
  #overlay.start-screen:before { content: ""; position: absolute; inset: 0; background: linear-gradient(180deg,rgba(5,7,10,.35),transparent 30%,transparent 72%,rgba(5,7,10,.72)); pointer-events: none; }
  #overlay.start-screen #panel { position: relative; width: 100%; min-height: 100%; padding: 48px clamp(28px,8vw,118px) 34px; display: flex; flex-direction: column; justify-content: space-between; border: 0; background: transparent; box-shadow: none; }
  #menu-content { min-height: calc(100vh - 82px); display: grid; grid-template-rows: auto 1fr auto; }
  .title-header { padding-top: 6px; }
  .title-index { color: rgba(232,237,242,.5); font-size: 9px; font-weight: 700; letter-spacing: .28em; }
  .title-index:before { content: ""; display: inline-block; width: 7px; height: 7px; margin-right: 10px; background: #f34d4d; box-shadow: 0 0 13px #f34d4d; }
  .title-header h1 { margin: 20px 0 8px; font-size: clamp(58px,10vw,112px); letter-spacing: -.075em; line-height: .78; text-shadow: 0 8px 40px #0009; }
  .title-tagline { color: rgba(231,235,239,.68); font-size: 10px; font-weight: 700; letter-spacing: .35em; }
  .start-layout { align-self: center; display: grid; grid-template-columns: minmax(240px,410px) minmax(230px,300px); align-items: end; gap: clamp(36px,10vw,170px); padding: 32px 0; }
  .start-pitch { max-width: 410px; }
  .start-pitch h2 { margin: 0 0 16px; font-size: clamp(28px,4vw,48px); line-height: 1.03; letter-spacing: -.055em; }
  .start-pitch h2 span { color: #f34d4d; }
  .start-pitch p { max-width: 370px; color: rgba(223,228,233,.69); font-size: 14px; line-height: 1.8; }
  .start-stamp { margin-top: 27px; padding-top: 14px; border-top: 1px solid rgba(232,237,242,.2); color: rgba(232,237,242,.46); font-size: 9px; letter-spacing: .2em; }
  .start-stamp b { color: #f1eee9; }
  .start-actions { padding: 20px; border: 1px solid rgba(230,235,240,.2); background: linear-gradient(145deg,rgba(10,13,18,.84),rgba(10,13,18,.57)); backdrop-filter: blur(9px); box-shadow: 0 22px 65px #0005; }
  .start-actions-label { margin-bottom: 15px; color: rgba(230,235,240,.55); font-size: 9px; font-weight: 700; letter-spacing: .24em; }
  .eyebrow { color: #f34d4d; font-size: 10px; font-weight: 800; letter-spacing: .35em; }
  h1 { margin: 15px 0 10px; font-size: clamp(42px, 9vw, 64px); line-height: .9; letter-spacing: -.06em; }
  h1 span { color: #f34d4d; }
  #message { color: #b7b3ae; font-size: 13px; line-height: 1.65; }
  #start { width: 100%; margin-top: 10px; padding: 15px; border: 0; background: #f34d4d; color: #120f0f; font-size: 11px; font-weight: 900; letter-spacing: .22em; cursor: pointer; }
  #start:hover { background: #ff7169; }
  #tutorial-open, .tutorial-nav button { padding: 13px; border: 1px solid #494644; background: transparent; color: #f1eee9; font-size: 10px; font-weight: 800; letter-spacing: .18em; cursor: pointer; }
  #tutorial-open { width: 100%; margin-top: 9px; }
  #tutorial-open:hover, .tutorial-nav button:hover { border-color: #f34d4d; }
  #tutorial { min-height: 260px; }
  #tutorial-heading { margin: 22px 0 12px; font-size: 28px; letter-spacing: -.04em; }
  #tutorial-copy { min-height: 82px; color: #b7b3ae; font-size: 14px; line-height: 1.7; }
  #tutorial-step { margin: 20px 0 12px; color: #929393; font-size: 10px; letter-spacing: .16em; }
  .tutorial-nav { display: flex; gap: 10px; }
  .tutorial-nav button { flex: 1; }
  .tutorial-nav #tutorial-next { border-color: #f34d4d; background: #f34d4d; color: #120f0f; }
  .keys { margin-top: 0; color: rgba(220,225,230,.58); font-size: 10px; line-height: 1.8; letter-spacing: .08em; }
  .keys b { color: #d3d0ca; font-weight: 600; }
  @media (max-width: 700px) { .topline { top: 22px; left: 24px; right: 24px; flex-direction: column; } .stats { align-self: stretch; justify-content: space-between; } .stat { min-width: 0; flex: 1; padding: 8px 6px; } .stat-value { font-size: 14px; } #replay-title { top: 128px; left: 24px; } #objective { left: 24px; bottom: 64px; width: 155px; } #motion-status { right: 24px; bottom: 64px; min-width: 0; } #hint { bottom: 17px; font-size: 8px; } #hud:after { inset: 8px; } #overlay.start-screen #panel { padding: 32px 30px 26px; } #menu-content { min-height: calc(100vh - 58px); } .title-header h1 { font-size: clamp(54px,13vw,82px); } .start-layout { grid-template-columns: minmax(0,1fr) minmax(210px,.8fr); gap: 25px; } .start-pitch p { font-size: 12px; } }
  @media (max-width: 520px) { .topline { left: 17px; right: 17px; } .stat { padding: 7px 4px; } .stat-label { font-size: 7px; letter-spacing: .12em; } .stat-value { font-size: 12px; } #objective { left: 17px; width: 135px; } #motion-status { right: 17px; } #hint { max-width: calc(100vw - 24px); overflow: hidden; } #overlay.start-screen #panel { padding: 28px 22px 22px; } #menu-content { min-height: calc(100vh - 50px); } .title-header h1 { margin-top: 16px; font-size: clamp(52px,16vw,76px); } .title-tagline { font-size: 8px; letter-spacing: .24em; } .start-layout { align-self: center; grid-template-columns: 1fr; gap: 16px; padding: 28px 0; } .start-pitch h2 { font-size: 30px; } .start-pitch p { margin: 8px 0; } .start-stamp { margin-top: 13px; padding-top: 9px; } .start-actions { padding: 14px; } .keys { font-size: 9px; } }
</style>
</head>
<body data-motion="normal" data-game-state="menu">
<canvas id="game"></canvas>
<div id="hud">
  <div id="replay-title">LEVEL REPLAY / PLAYER CAMERA</div>
  <div id="freecam-label">FREE CAMERA &nbsp;·&nbsp; MOUSE LOOK &nbsp;·&nbsp; WASD FLY &nbsp;·&nbsp; SPACE / CTRL UP / DOWN &nbsp;·&nbsp; SHIFT BOOST &nbsp;·&nbsp; F RETURN</div>
  <div class="topline">
    <div>
      <div class="brand">STILL <span>/</span> RED</div>
      <div class="mission"><i></i>FIELD OPERATION &nbsp;·&nbsp; <span id="level">ROOM 01</span></div>
    </div>
    <div class="stats">
      <div class="stat"><span class="stat-label">HOSTILES</span><span class="stat-value" id="hostiles">05</span></div>
      <div class="stat"><span class="stat-label">CLEARED</span><span class="stat-value" id="score">00</span></div>
      <div class="stat"><span class="stat-label">LOADOUT</span><span class="stat-value" id="weapon">SIDEARM</span></div>
      <div class="stat"><span class="stat-label">ELAPSED</span><span class="stat-value" id="timer">00:00</span></div>
    </div>
  </div>
  <div id="objective">
    <div id="objective-label">PRIMARY OBJECTIVE</div>
    <div id="objective-copy">ELIMINATE HOSTILES</div>
    <div id="objective-meter"><div id="objective-fill"></div></div>
  </div>
  <div id="motion-status" aria-live="polite">
    <div id="motion-indicator"></div>
    <div><div id="motion-copy">KINETIC / NORMAL</div><div id="motion-detail">MOTION AT FULL SPEED</div></div>
  </div>
  <div id="crosshair"><i></i></div>
  <div id="toast">SHOT FIRED</div>
  <div id="hint">WASD MOVE &nbsp;·&nbsp; SPACE JUMP &nbsp;·&nbsp; MOUSE AIM &nbsp;·&nbsp; CLICK FIRE / PUNCH UNARMED &nbsp;·&nbsp; Q POSSESS &nbsp;·&nbsp; E PICK UP &nbsp;·&nbsp; P HITBOXES &nbsp;·&nbsp; C ACCESS &nbsp;·&nbsp; L LEVEL SELECT &nbsp;·&nbsp; F FREE CAM</div>
</div>
<div id="overlay" class="start-screen">
  <section id="panel">
    <div id="menu-content">
      <header class="title-header">
        <div class="title-index">A SLOW-MOTION SURVIVAL GAME</div>
        <h1>STILL<span>/</span><br>RED</h1>
        <div class="title-tagline">WHEN YOU STOP, THEY SLOW.</div>
      </header>
      <div class="start-layout">
        <div class="start-pitch">
          <h2>THE ROOM IS<br><span>ALREADY MOVING.</span></h2>
          <p>Fight through 15 locations, from local shops and offices to a hotel, museum, and skyline tower. The last seven levels have two stories connected by climbable stairs. Red security is quick when you are. Stop moving and time crawls.</p>
          <div class="start-stamp"><b>15 LOCATIONS</b> &nbsp;·&nbsp; ONE RULE &nbsp;·&nbsp; NO WAY OUT</div>
        </div>
        <div class="start-actions">
          <div class="start-actions-label">FIELD BRIEFING &nbsp; / &nbsp; 01</div>
          <button id="start">ENTER THE ROOM</button>
          <button id="tutorial-open" type="button">HOW TO PLAY</button>
        </div>
      </div>
      <div class="keys"><b>W A S D</b> move &nbsp; <b>SPACE</b> jump &nbsp; <b>MOUSE</b> aim &nbsp; <b>LEFT CLICK</b> fire<br><b>Q</b> possess &nbsp; <b>E</b> pick up &nbsp; <b>P</b> enemy hitboxes &nbsp; <b>C</b> secret access &nbsp; <b>L</b> level select</div>
    </div>
    <div id="tutorial" hidden aria-live="polite">
      <div class="eyebrow">FIELD GUIDE</div>
      <h2 id="tutorial-heading"></h2>
      <p id="tutorial-copy"></p>
      <div id="tutorial-step"></div>
      <div class="tutorial-nav">
        <button id="tutorial-back" type="button">BACK</button>
        <button id="tutorial-next" type="button">NEXT</button>
      </div>
    </div>
  </section>
</div>
<div id="cheat-menu" hidden role="dialog" aria-modal="true" aria-labelledby="cheat-heading">
  <section id="cheat-panel">
    <div class="eyebrow">FIELD ACCESS / RESTRICTED</div>
    <h2 id="cheat-heading">SECRET ACCESS</h2>
    <p>Enter an authorization code to access experimental field controls.</p>
    <form id="cheat-code-form">
      <input id="cheat-code" type="text" inputmode="text" maxlength="24" autocomplete="off" aria-label="Secret code" placeholder="ENTER CODE">
      <button type="submit">UNLOCK</button>
    </form>
    <p id="cheat-message" aria-live="polite"></p>
    <div id="cheat-options" hidden>
      <label><input id="wallhack-toggle" type="checkbox"> WALL VISION — reveal operatives through walls</label>
      <label><input id="sword-toggle" type="checkbox"> SWORD — equip the melee weapon</label>
      <label><input id="no-reload-toggle" type="checkbox"> NO RELOAD — remove weapon cooldown</label>
    </div>
    <button id="cheat-close" type="button">CLOSE / RESUME</button>
  </section>
</div>
<div id="settings-menu" hidden role="dialog" aria-modal="true" aria-labelledby="settings-heading">
  <section id="settings-panel">
    <div class="eyebrow">FIELD CONFIGURATION</div>
    <h2 id="settings-heading">SETTINGS</h2>
    <label class="setting-row" for="sensitivity-setting">
      <span class="setting-label"><span>MOUSE SENSITIVITY</span><output id="sensitivity-value" for="sensitivity-setting">100%</output></span>
      <input id="sensitivity-setting" type="range" min="0.5" max="3" step="0.1" value="1">
      <span class="setting-note">Adjust how quickly the camera turns with mouse movement.</span>
    </label>
    <label class="setting-row setting-label" for="palette-toggle">
      <span>SWAP BLACK / WHITE SCHEME</span>
      <input id="palette-toggle" type="checkbox">
    </label>
    <p class="setting-note">Press ESC to close and resume.</p>
    <button id="settings-close" type="button">CLOSE / RESUME</button>
  </section>
</div>
<div id="level-select" hidden role="dialog" aria-modal="true" aria-labelledby="level-select-heading">
  <section id="level-select-panel">
    <div class="eyebrow">CAMPAIGN / MISSION SELECT</div>
    <h2 id="level-select-heading">CHOOSE A ROOM</h2>
    <p id="level-progress" aria-live="polite"></p>
    <div id="level-grid"></div>
    <button id="level-select-close" type="button">CLOSE / RESUME</button>
  </section>
</div>
<script>
(() => {
  const canvas = document.querySelector("#game");
  const gl = canvas.getContext("webgl", { antialias: true, alpha: false });
  if (!gl) {
    document.querySelector("#message").textContent = "This game needs a browser with WebGL enabled.";
    document.querySelector("#start").disabled = true;
    return;
  }

  const vertexSource = `
    attribute vec3 aPosition;
    uniform mat4 uMvp;
    void main() { gl_Position = uMvp * vec4(aPosition, 1.0); }
  `;
  const fragmentSource = `
    precision mediump float;
    uniform vec3 uColor;
    void main() { gl_FragColor = vec4(uColor, 1.0); }
  `;
  function shader(type, source) {
    const result = gl.createShader(type);
    gl.shaderSource(result, source);
    gl.compileShader(result);
    if (!gl.getShaderParameter(result, gl.COMPILE_STATUS)) throw new Error(gl.getShaderInfoLog(result));
    return result;
  }
  const program = gl.createProgram();
  gl.attachShader(program, shader(gl.VERTEX_SHADER, vertexSource));
  gl.attachShader(program, shader(gl.FRAGMENT_SHADER, fragmentSource));
  gl.linkProgram(program);
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) throw new Error(gl.getProgramInfoLog(program));
  gl.useProgram(program);

  const cubeVertices = new Float32Array([
    -1,-1, 1,  1,-1, 1,  1, 1, 1,  -1,-1, 1,  1, 1, 1, -1, 1, 1,
     1,-1,-1, -1,-1,-1, -1, 1,-1,   1,-1,-1, -1, 1,-1,  1, 1,-1,
    -1,-1,-1, -1,-1, 1, -1, 1, 1,  -1,-1,-1, -1, 1, 1, -1, 1,-1,
     1,-1, 1,  1,-1,-1,  1, 1,-1,   1,-1, 1,  1, 1,-1,  1, 1, 1,
    -1, 1, 1,  1, 1, 1,  1, 1,-1,  -1, 1, 1,  1, 1,-1, -1, 1,-1,
    -1,-1,-1,  1,-1,-1,  1,-1, 1,  -1,-1,-1,  1,-1, 1, -1,-1, 1
  ]);
  const buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
  gl.bufferData(gl.ARRAY_BUFFER, cubeVertices, gl.STATIC_DRAW);
  const position = gl.getAttribLocation(program, "aPosition");
  gl.enableVertexAttribArray(position);
  gl.vertexAttribPointer(position, 3, gl.FLOAT, false, 0, 0);
  const sphereVertices = [];
  const sphereSegments = 16, sphereRings = 12;
  for (let ring=0;ring<sphereRings;ring++) {
    const top=Math.PI*ring/sphereRings, bottom=Math.PI*(ring+1)/sphereRings;
    for (let segment=0;segment<sphereSegments;segment++) {
      const left=Math.PI*2*segment/sphereSegments, right=Math.PI*2*(segment+1)/sphereSegments;
      const point=(phi,theta)=>[Math.sin(phi)*Math.cos(theta),Math.cos(phi),Math.sin(phi)*Math.sin(theta)];
      const a=point(top,left), b=point(bottom,left), c=point(bottom,right), d=point(top,right);
      sphereVertices.push(...a,...b,...c,...a,...c,...d);
    }
  }
  const sphereBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,sphereBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(sphereVertices),gl.STATIC_DRAW);
  const sphereVertexCount=sphereVertices.length/3;
  const barrelVertices=[];
  const barrelSegments=16;
  for(let segment=0;segment<barrelSegments;segment++) {
    const a=segment*Math.PI*2/barrelSegments,b=(segment+1)*Math.PI*2/barrelSegments;
    const nearA=[Math.cos(a),Math.sin(a),-.5],nearB=[Math.cos(b),Math.sin(b),-.5];
    const farA=[Math.cos(a),Math.sin(a),.5],farB=[Math.cos(b),Math.sin(b),.5];
    barrelVertices.push(...nearA,...nearB,...farB,...nearA,...farB,...farA);
    barrelVertices.push(0,0,.5,...farA,...farB,0,0,-.5,...nearB,...nearA);
  }
  const barrelBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,barrelBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(barrelVertices),gl.STATIC_DRAW);
  const barrelVertexCount=barrelVertices.length/3;
  const swordBladeVertices=[];
  const addBladeTriangle=(a,b,c)=>swordBladeVertices.push(...a,...b,...c,...c,...b,...a);
  const bladeLeftBase=[-.075,0,-.06],bladeRightBase=[.075,0,-.06];
  const bladeLeftShoulder=[-.036,0,-.78],bladeRightShoulder=[.036,0,-.78];
  const bladeTip=[0,0,-1.02];
  const bladeTopBaseLeft=[-.075,.014,-.06],bladeTopBaseRight=[.075,.014,-.06];
  const bladeTopShoulderLeft=[-.036,.009,-.78],bladeTopShoulderRight=[.036,.009,-.78];
  addBladeTriangle(bladeTopBaseLeft,bladeTopBaseRight,bladeTip);
  addBladeTriangle(bladeTopBaseLeft,bladeTip,bladeTopShoulderLeft);
  addBladeTriangle(bladeTopBaseRight,bladeTopShoulderRight,bladeTip);
  addBladeTriangle([bladeLeftBase[0],-.014,bladeLeftBase[2]],[bladeRightBase[0],-.014,bladeRightBase[2]],bladeTip);
  addBladeTriangle([bladeLeftBase[0],-.014,bladeLeftBase[2]],bladeTip,[bladeLeftShoulder[0],-.009,bladeLeftShoulder[2]]);
  addBladeTriangle([bladeRightBase[0],-.014,bladeRightBase[2]],[bladeRightShoulder[0],-.009,bladeRightShoulder[2]],bladeTip);
  addBladeTriangle(bladeLeftBase,bladeTopBaseLeft,[bladeTopShoulderLeft[0],bladeTopShoulderLeft[1],bladeTopShoulderLeft[2]]);
  addBladeTriangle(bladeLeftBase,[bladeTopShoulderLeft[0],bladeTopShoulderLeft[1],bladeTopShoulderLeft[2]],bladeLeftShoulder);
  addBladeTriangle(bladeRightBase,bladeRightShoulder,[bladeTopShoulderRight[0],bladeTopShoulderRight[1],bladeTopShoulderRight[2]]);
  addBladeTriangle(bladeRightBase,[bladeTopShoulderRight[0],bladeTopShoulderRight[1],bladeTopShoulderRight[2]],bladeTopBaseRight);
  const swordBladeBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,swordBladeBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(swordBladeVertices),gl.STATIC_DRAW);
  const swordBladeVertexCount=swordBladeVertices.length/3;
  const throwingStarVertices=[];
  for(let i=0;i<16;i++) {
    const angle=i*Math.PI/8, radius=i%2===0?.28:.085;
    const nextAngle=(i+1)*Math.PI/8, nextRadius=(i+1)%2===0?.28:.085;
    const a=[Math.cos(angle)*radius,Math.sin(angle)*radius];
    const b=[Math.cos(nextAngle)*nextRadius,Math.sin(nextAngle)*nextRadius];
    throwingStarVertices.push(0,0,.02,a[0],a[1],.02,b[0],b[1],.02);
    throwingStarVertices.push(0,0,-.02,b[0],b[1],-.02,a[0],a[1],-.02);
  }
  const throwingStarBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,throwingStarBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(throwingStarVertices),gl.STATIC_DRAW);
  const throwingStarVertexCount=throwingStarVertices.length/3;
  const cupVertices=[];
  const cupSegments=16;
  for(let i=0;i<cupSegments;i++) {
    const a=i*Math.PI*2/cupSegments,b=(i+1)*Math.PI*2/cupSegments;
    const bottomA=[Math.cos(a)*.72,-1,Math.sin(a)*.72];
    const bottomB=[Math.cos(b)*.72,-1,Math.sin(b)*.72];
    const topA=[Math.cos(a),1,Math.sin(a)];
    const topB=[Math.cos(b),1,Math.sin(b)];
    cupVertices.push(...bottomA,...topB,...bottomB,...bottomA,...topA,...topB);
    cupVertices.push(0,-1,0,...bottomA,...bottomB);
  }
  const cupBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,cupBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(cupVertices),gl.STATIC_DRAW);
  const cupVertexCount=cupVertices.length/3;
  const hitboxVertices=new Float32Array([
    -1,-1,-1, 1,-1,-1,  1,-1,-1, 1,1,-1,  1,1,-1, -1,1,-1,  -1,1,-1, -1,-1,-1,
    -1,-1,1, 1,-1,1,  1,-1,1, 1,1,1,  1,1,1, -1,1,1,  -1,1,1, -1,-1,1,
    -1,-1,-1, -1,-1,1,  1,-1,-1, 1,-1,1,  1,1,-1, 1,1,1,  -1,1,-1, -1,1,1
  ]);
  const hitboxBuffer=gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER,hitboxBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,hitboxVertices,gl.STATIC_DRAW);
  const mvpLocation = gl.getUniformLocation(program, "uMvp");
  const colorLocation = gl.getUniformLocation(program, "uColor");
  gl.enable(gl.DEPTH_TEST);
  gl.enable(gl.CULL_FACE);

  const identity = () => new Float32Array([1,0,0,0, 0,1,0,0, 0,0,1,0, 0,0,0,1]);
  function multiply(a, b) {
    const out = new Float32Array(16);
    for (let c = 0; c < 4; c++) for (let r = 0; r < 4; r++)
      out[c * 4 + r] = a[r] * b[c * 4] + a[4 + r] * b[c * 4 + 1] + a[8 + r] * b[c * 4 + 2] + a[12 + r] * b[c * 4 + 3];
    return out;
  }
  function translate(x, y, z) {
    const m = identity(); m[12] = x; m[13] = y; m[14] = z; return m;
  }
  function scale(x, y, z) {
    const m = identity(); m[0] = x; m[5] = y; m[10] = z; return m;
  }
  function rotateY(a) {
    const c = Math.cos(a), s = Math.sin(a);
    return new Float32Array([c,0,-s,0, 0,1,0,0, s,0,c,0, 0,0,0,1]);
  }
  function rotateX(a) {
    const c = Math.cos(a), s = Math.sin(a);
    return new Float32Array([1,0,0,0, 0,c,s,0, 0,-s,c,0, 0,0,0,1]);
  }
  function rotateZ(a) {
    const c = Math.cos(a), s = Math.sin(a);
    return new Float32Array([c,s,0,0, -s,c,0,0, 0,0,1,0, 0,0,0,1]);
  }
  function perspective(fov, aspect, near, far) {
    const f = 1 / Math.tan(fov / 2), nf = 1 / (near - far);
    return new Float32Array([f/aspect,0,0,0, 0,f,0,0, 0,0,(far+near)*nf,-1, 0,0,2*far*near*nf,0]);
  }
  function lookAt(eye, target) {
    let zx = eye[0]-target[0], zy = eye[1]-target[1], zz = eye[2]-target[2];
    let length = Math.hypot(zx,zy,zz); zx/=length; zy/=length; zz/=length;
    let xx = zz, xy = 0, xz = -zx;
    length = Math.hypot(xx,xy,xz); xx/=length; xy/=length; xz/=length;
    const yx = zy*xz-zz*xy, yy = zz*xx-zx*xz, yz = zx*xy-zy*xx;
    return new Float32Array([
      xx,yx,zx,0, xy,yy,zy,0, xz,yz,zz,0,
      -(xx*eye[0]+xy*eye[1]+xz*eye[2]), -(yx*eye[0]+yy*eye[1]+yz*eye[2]), -(zx*eye[0]+zy*eye[1]+zz*eye[2]), 1
    ]);
  }
  const levels = [
    {name:"HOUSE",layout:"house",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["mug",-2.2,-4,.48],["cup",-6,-5,.96],["tumbler",5.5,-8.7,1.3]],blocks:[
      {x:-3.6,z:-1.6,w:1.2,d:2.4},{x:3.3,z:1.5,w:1.2,d:2.6},
      {x:0,z:-4.8,w:3.4,d:.65},{x:-5.5,z:3.7,w:2.2,d:.7},{x:5.2,z:-3.6,w:1.7,d:.7}
    ], pickups:[["shotgun",2,6],["sword",-2,6],["ar",0,4.7]], enemies:[[-7,1],[-5,-6],[0,-8],[6,-5],[7,4]]},
    {name:"CORNER STORE",layout:"store",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["cup",-7.2,-6.7,1.38],["tumbler",7.7,5.8,1.02],["mug",-5,-2,1.8]],blocks:[
      {x:-4.3,z:-3.3,w:1.1,d:3.1},{x:4.1,z:-1.8,w:1.1,d:3.3},
      {x:-1.8,z:1.1,w:2.7,d:.75},{x:2.8,z:4.1,w:2.4,d:.75},
      {x:-6,z:4.3,w:1.6,d:.7},{x:6.4,z:-5,w:1.4,d:.7}
    ], pickups:[["shotgun",2,7],["sword",-2,7],["ar",0,5.8]], enemies:[[-7,-6],[-6,1],[-2,-7],[2,-7],[7,-5],[7,3]]},
    {name:"NEIGHBORHOOD BAR",layout:"bar",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["mug",-2,-6.6,1.18],["cup",1,-6.6,1.18],["tumbler",4,-6.6,1.18]],blocks:[
      {x:-5.3,z:-1.1,w:1,d:3.7},{x:5.1,z:1.1,w:1,d:3.7},
      {x:-2.2,z:-5.1,w:2.7,d:.7},{x:2.5,z:-2.7,w:2.5,d:.7},
      {x:-2.4,z:2.4,w:2.4,d:.7},{x:2.2,z:5,w:2.8,d:.7},
      {x:0,z:-.15,w:1.1,d:1.1}
    ], pickups:[["shotgun",1,7.3],["sword",-1,7.3],["ar",0,6.1]], enemies:[[-8,-7],[-7,3],[-3,-7],[0,-8],[4,-7],[7,-3],[7,6]]},
    {name:"WAREHOUSE",layout:"warehouse",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["tumbler",-6,-7.8,1.28],["mug",2,7.6,1.28],["cup",6,7.6,1.28]],blocks:[
      {x:-6,z:-1,w:1,d:5.2},{x:6,z:1,w:1,d:5.2},
      {x:-2.8,z:-5.1,w:3.5,d:.7},{x:2.8,z:5.2,w:3.5,d:.7},
      {x:-2.2,z:-.8,w:2.4,d:.7},{x:2.2,z:1.1,w:2.4,d:.7},
      {x:0,z:-3.2,w:1.4,d:.65}
    ], pickups:[], enemies:[[-8,-7],[-4,-8],[0,-8],[4,-8],[8,-6],[-8,4],[-4,1],[4,-2],[8,5]]},
    {name:"OFFICE",layout:"office",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["mug",-6,-4,1.04],["cup",0,-4,1.04],["tumbler",6,4,1.04]],blocks:[
      {x:-4.8,z:-4,w:3.5,d:.8},{x:-1,z:-4,w:.8,d:3.4},
      {x:3,z:-5,w:3.2,d:.8},{x:5.4,z:-1.5,w:.8,d:3.2},
      {x:1.7,z:1.4,w:3.6,d:.8},{x:-2.3,z:2.4,w:.8,d:3.2},
      {x:-5.4,z:5,w:2.8,d:.8},{x:2.7,z:6,w:3.8,d:.8}
    ], pickups:[], enemies:[[-8,-8],[-4,-1],[0,-7],[4,-3],[8,-7],[7,3],[-7,7],[0,4],[5,7],[-4,5]]},
    {name:"SUBWAY STATION",layout:"station",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],props:[["cup",-5,5,.85],["tumbler",0,5,.85],["mug",5,5,.85]],blocks:[
      {x:-5.7,z:-1,w:1,d:6.4},{x:5.7,z:1,w:1,d:6.4},
      {x:0,z:-5.6,w:6.5,d:.8},{x:0,z:5.6,w:6.5,d:.8},
      {x:-2.5,z:-2.6,w:2.3,d:.75},{x:2.5,z:2.6,w:2.3,d:.75},
      {x:-2.5,z:2.6,w:2.3,d:.75},{x:2.5,z:-2.6,w:2.3,d:.75},
      {x:0,z:0,w:1,d:1}
    ], pickups:[], enemies:[[-8,-8],[-4,-8],[0,-8],[4,-8],[8,-8],[-8,0],[8,0],[-8,8],[-4,8],[0,8],[4,8],[8,8]]},
    {name:"MOTEL LOBBY",layout:"house",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-5,z:-4,w:2.8,d:.8},{x:4,z:-4,w:.8,d:3.2},{x:-3,z:2,w:3,d:.8},{x:4,z:4,w:2.6,d:.8}
    ],pickups:[["shotgun",2,7]],enemies:[[-8,-7],[-4,-6],[0,-8],[5,-7],[8,-2],[-7,2],[-2,5],[4,7],[8,6]]},
    {name:"FOOD COURT",layout:"store",floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:-1,w:1,d:5},{x:6,z:1,w:1,d:5},{x:-2,z:-4,w:2.8,d:.8},{x:2,z:4,w:2.8,d:.8},{x:0,z:0,w:1.6,d:1.6}
    ],pickups:[["ar",-2,7]],enemies:[[-8,-8],[-4,-7],[0,-8],[4,-7],[8,-8],[-8,3],[-4,7],[2,6],[7,4],[8,-2]]},
    {name:"TOWNHOUSE",layout:"house",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:1,w:2.6,d:.8},{x:3,z:2,w:.8,d:3},{x:5,z:6,w:2.5,d:.8}
    ],upperBlocks:[{x:-6,z:-7,w:2.8,d:.8},{x:2,z:-7,w:3,d:.8},{x:6,z:-5,w:.8,d:2.4},{x:4,z:-4,w:2.4,d:.8}],pickups:[["shotgun",5,6],["ar",6,-7]],enemies:[[-7,6],[1,1],[7,7],[-7,3],[6,2],[-8,-5],[-5,-8],[0,-8],[5,-8],[8,-5]]},
    {name:"DEPARTMENT STORE",layout:"store",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:-1,w:1,d:5},{x:6,z:1,w:1,d:5},{x:-1,z:5,w:3,d:.8},{x:2,z:2,w:2,d:.8}
    ],upperBlocks:[{x:-6,z:-7,w:1,d:4},{x:-3,z:-6,w:2.8,d:.8},{x:3,z:-6,w:2.8,d:.8},{x:6,z:-7,w:1,d:4}],pickups:[["shotgun",7,7],["sword",7,-8]],enemies:[[-8,7],[-5,7],[1,5],[5,7],[8,2],[-8,-8],[-4,-8],[1,-7],[5,-8],[8,-5]]},
    {name:"GRAND HOTEL",layout:"office",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-5,z:0,w:3,d:.8},{x:5,z:2,w:3,d:.8},{x:0,z:5,w:4,d:.8}
    ],upperBlocks:[{x:-6,z:-6,w:2.4,d:.8},{x:-3,z:-8,w:.8,d:2.8},{x:4,z:-7,w:3,d:.8},{x:7,z:-5,w:.8,d:2.4}],pickups:[["ar",-7,7],["shotgun",7,-8]],enemies:[[-8,7],[-3,7],[3,7],[8,7],[-7,2],[7,1],[-8,-8],[-4,-8],[2,-8],[7,-7]]},
    {name:"IRONWORKS",layout:"warehouse",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:0,w:1,d:5},{x:6,z:0,w:1,d:5},{x:0,z:5,w:4,d:.8},{x:0,z:-1,w:2,d:2}
    ],upperBlocks:[{x:-7,z:-7,w:1,d:4},{x:-4,z:-7,w:2.4,d:.8},{x:1,z:-6,w:3,d:.8},{x:6,z:-7,w:1,d:4}],pickups:[["shotgun",7,7],["ar",7,-8]],enemies:[[-8,7],[-4,7],[1,1],[5,7],[8,1],[-8,-8],[-5,-7],[0,-8],[4,-8],[8,-6]]},
    {name:"POLICE STATION",layout:"office",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:1,w:2.6,d:.8},{x:-2,z:4,w:2.4,d:.8},{x:4,z:2,w:3,d:.8},{x:7,z:6,w:.8,d:2}
    ],upperBlocks:[{x:-6,z:-7,w:3,d:.8},{x:-2,z:-5,w:.8,d:2.6},{x:2,z:-8,w:3,d:.8},{x:6,z:-5,w:.8,d:2.8}],pickups:[["sword",7,7],["shotgun",7,-8]],enemies:[[-8,7],[-4,7],[1,1],[5,6],[8,1],[-8,-8],[-5,-7],[0,-8],[4,-8],[8,-5]]},
    {name:"CITY MUSEUM",layout:"station",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:0,w:1,d:5},{x:6,z:0,w:1,d:5},{x:-3,z:3,w:2,d:1},{x:3,z:-1,w:2,d:1},{x:0,z:6,w:2,d:.8}
    ],upperBlocks:[{x:-6,z:-7,w:1,d:4},{x:-3,z:-6,w:2,d:1},{x:1,z:-8,w:2,d:1},{x:5,z:-6,w:2,d:1},{x:7,z:-7,w:1,d:3}],pickups:[["ar",7,7],["shotgun",7,-8]],enemies:[[-8,7],[-4,7],[1,1],[5,7],[8,1],[-8,-8],[-4,-8],[0,-8],[4,-8],[8,-5]]},
    {name:"SKYLINE TOWER",layout:"bar",twoStory:true,floor:[.9,.9,.9],wall:[.98,.98,.98],trim:[.82,.82,.82],cover:[.88,.88,.88],blocks:[
      {x:-6,z:-1,w:1,d:4},{x:6,z:1,w:1,d:4},{x:-2,z:4,w:2.4,d:.8},{x:3,z:5,w:2.5,d:.8}
    ],upperBlocks:[{x:-6,z:-7,w:1,d:4},{x:-3,z:-7,w:2.5,d:.8},{x:2,z:-5,w:.8,d:3},{x:5,z:-8,w:2.6,d:.8},{x:7,z:-5,w:.8,d:3}],pickups:[["sword",7,7],["ar",7,-8]],enemies:[[-8,7],[-4,7],[1,1],[4,7],[8,6],[-8,-8],[-5,-7],[0,-8],[4,-8],[8,-5]]},
  ];
  const progressStorageKey="still-red-unlocked-levels";
  let unlockedLevelCount=1;
  try {
    const savedProgress=Number.parseInt(localStorage.getItem(progressStorageKey)||"1",10);
    if(Number.isInteger(savedProgress)) unlockedLevelCount=Math.max(1,Math.min(levels.length,savedProgress));
  } catch(error) {
    console.warn("Could not load campaign progress.",error);
  }
  let blocks = [];
  const player = { x: 0, y: 0, verticalVelocity: 0, z: 7.3, yaw: 0, pitch: 0 };
  const freeCamera={active:false,x:0,y:0,z:0,yaw:0,pitch:0};
  let enemies = [], bullets = [], thrownGun = null, thrownProps = [], droppedGuns = [], shards = [], explosions = [], weaponPickups = [], props = [], heldProp = null, hasGun = true;
  let weaponType = "sidearm";
  let lastShotAt = -Infinity;
  let swordSwingUntil = 0;
  let punchSwingUntil=0, lastPunchAt=-Infinity;
  let cheatsUnlocked = false, wallHackEnabled = false, swordCheatEnabled = false, noReloadEnabled = false, onePunchEnabled = false, throwingStarsUnlocked=false, secretWeaponType=null;
  let mouseSensitivity=1, swappedPalette=false;
  let swordRestore = null, cheatMenuWasRunning = false;
  let lastPossessAt = -Infinity;
  let levelIndex = 0, betweenLevels = false, finishPending = false, finishDeadline = 0;
  let levelSelectWasRunning=false;
  const replayHistory=[];
  let replayFrames=[], replayLastSample=0, replayStartedAt=0;
  let replayPlaybackTimes=[], replayPlaybackDuration=0;
  const replaySampleInterval=1000/120, replayHistoryLimit=601;
  const replayNormalSpeed=.5, replaySlowMotionSpeed=2;
  const swordSwingDuration=540;
  const enemyHitboxHalfWidth=.48, enemyHitboxHalfHeight=1.02, enemyHitboxHalfDepth=.34;
  const swordSwingKeyframes=[
    [0,0,0,0,0],
    [.12,.66,.68,.12,.1],
    [.5,-.92,-.72,-.34,-.1],
    [.8,-.32,-.22,-.16,-.04],
    [1,0,0,0,0]
  ];
  function smoothStep(value) {
    return value*value*(3-2*value);
  }
  let showEnemyHitboxes = false;
  let score = 0, elapsed = 0, last = 0, running = false, won = false, ended = false;
  const held = new Set();
  const overlay = document.querySelector("#overlay");
  const replayTitle = document.querySelector("#replay-title");
  const hint = document.querySelector("#hint");
  const startButton = document.querySelector("#start");
  const menuContent = document.querySelector("#menu-content");
  const tutorial = document.querySelector("#tutorial");
  const tutorialHeading = document.querySelector("#tutorial-heading");
  const tutorialCopy = document.querySelector("#tutorial-copy");
  const tutorialStep = document.querySelector("#tutorial-step");
  let overlayAction=null;
  const tutorialPages = [
    {heading:"Move and jump",copy:"Use W, A, S, D to move around the arena. Press SPACE to jump over danger. Keep moving to dodge incoming fire."},
    {heading:"Aim and attack",copy:"Move the mouse to aim and click to fire. If you have no gun, click to punch nearby operatives. Your weapon and punch have separate cooldowns. Use cover to block bullets."},
    {heading:"Use slow motion",copy:"When you stand still, the world slows down—including enemy movement and bullets. Moving or jumping returns the action to normal speed."},
    {heading:"Take over an operative",copy:"Aim at a red operative and press Q to take over their body. The targeted operative is eliminated, your old body explodes, and you inherit their weapon. Body takeover has a 5-second cooldown."},
    {heading:"Pick up and throw",copy:"Walk near a weapon or dark cup and press E to pick it up. Right-click to throw the equipped weapon or held cup. Thrown cups can knock out an operative and be picked up again."},
    {heading:"Clear the room",copy:"Eliminate every hostile to win. Press P to toggle enemy hitboxes, L to choose any room you have unlocked, or F for a free camera you can fly with WASD. Clearing a room unlocks the next one, and progress is saved in this browser."}
  ];
  let tutorialIndex=0;
  const hostiles = document.querySelector("#hostiles");
  const scoreText = document.querySelector("#score");
  const levelText = document.querySelector("#level");
  const objectiveCopy = document.querySelector("#objective-copy");
  const objectiveFill = document.querySelector("#objective-fill");
  const motionCopy = document.querySelector("#motion-copy");
  const motionDetail = document.querySelector("#motion-detail");
  const toast = document.querySelector("#toast");
  let toastTimer = 0;
  let mouseFiring = false;
  let audioContext = null;
  const cheatMenu = document.querySelector("#cheat-menu");
  const cheatCodeForm = document.querySelector("#cheat-code-form");
  const cheatCodeInput = document.querySelector("#cheat-code");
  const cheatMessage = document.querySelector("#cheat-message");
  const cheatOptions = document.querySelector("#cheat-options");
  const wallHackToggle = document.querySelector("#wallhack-toggle");
  const swordToggle = document.querySelector("#sword-toggle");
  const noReloadToggle = document.querySelector("#no-reload-toggle");
  const settingsMenu=document.querySelector("#settings-menu");
  const sensitivitySetting=document.querySelector("#sensitivity-setting");
  const sensitivityValue=document.querySelector("#sensitivity-value");
  const paletteToggle=document.querySelector("#palette-toggle");
  const settingsClose=document.querySelector("#settings-close");
  let settingsWasRunning=false,settingsPreviousState="menu";
  const levelSelect = document.querySelector("#level-select");
  const levelGrid = document.querySelector("#level-grid");
  try {
    const storedSensitivity=Number.parseFloat(localStorage.getItem("still-red-mouse-sensitivity")||"1");
    if(Number.isFinite(storedSensitivity)) mouseSensitivity=Math.max(.5,Math.min(3,storedSensitivity));
    swappedPalette=localStorage.getItem("still-red-swapped-palette")==="true";
  } catch(error) {
    console.warn("Could not load game settings.",error);
  }
  sensitivitySetting.value=String(mouseSensitivity);
  sensitivityValue.value=Math.round(mouseSensitivity*100)+"%";
  paletteToggle.checked=swappedPalette;
  function sceneColor(color) {
    if(!swappedPalette || Math.max(...color)-Math.min(...color)>=.12) return color;
    return color.map(channel=>1-channel);
  }
  function openSettings() {
    settingsWasRunning=running;
    settingsPreviousState=document.body.dataset.gameState||"menu";
    if(running) {
      running=false;
      mouseFiring=false;
      if(document.pointerLockElement===canvas) document.exitPointerLock();
    }
    settingsMenu.hidden=false;
    document.body.dataset.gameState="settings";
    settingsClose.focus();
  }
  function closeSettings() {
    if(settingsMenu.hidden) return;
    settingsMenu.hidden=true;
    if(settingsWasRunning && !finishPending && !ended) {
      running=true;
      last=performance.now();
      document.body.dataset.gameState="playing";
      canvas.requestPointerLock?.();
    } else {
      document.body.dataset.gameState=settingsPreviousState;
    }
    settingsWasRunning=false;
  }
  const levelProgress = document.querySelector("#level-progress");

  function showTutorialPage(index) {
    tutorialIndex=index;
    const page=tutorialPages[tutorialIndex];
    document.querySelector("#tutorial .eyebrow").textContent="FIELD GUIDE";
    tutorialHeading.textContent=page.heading;
    tutorialCopy.textContent=page.copy;
    tutorialStep.textContent=String(tutorialIndex+1).padStart(2,"0")+" / "+String(tutorialPages.length).padStart(2,"0");
    document.querySelector("#tutorial-back").hidden=false;
    document.querySelector("#tutorial-back").textContent=tutorialIndex===0?"MENU":"BACK";
    document.querySelector("#tutorial-next").textContent=tutorialIndex===tutorialPages.length-1?"START PLAYING":"NEXT";
  }
  function showOverlayMessage(label,heading,copy,buttonText,action) {
    overlayAction=action;
    overlay.classList.remove("start-screen");
    overlay.style.display="grid";
    menuContent.hidden=true;
    tutorial.hidden=false;
    document.querySelector("#tutorial .eyebrow").textContent=label;
    tutorialHeading.textContent=heading;
    tutorialCopy.textContent=copy;
    tutorialStep.textContent="";
    document.querySelector("#tutorial-back").hidden=true;
    document.querySelector("#tutorial-next").textContent=buttonText;
  }
  function setSwordCheat(enabled) {
    if(enabled===swordCheatEnabled) return;
    if(enabled) {
      swordRestore={weapon:weaponType,hasGun};
      swordCheatEnabled=true;
      weaponType="sword";
      hasGun=true;
    } else {
      swordCheatEnabled=false;
      if(weaponType==="sword") {
        weaponType=swordRestore?.weapon||"sidearm";
        hasGun=swordRestore?.hasGun??true;
      }
      swordRestore=null;
    }
    updateHud();
  }
  function openCheatMenu() {
    cheatMenuWasRunning=running;
    if(running) {
      running=false;
      mouseFiring=false;
      if(document.pointerLockElement===canvas) document.exitPointerLock();
    }
    cheatCodeForm.hidden=false;
    cheatOptions.hidden=!cheatsUnlocked;
    cheatCodeInput.value="";
    cheatMessage.textContent=cheatsUnlocked?"ACCESS GRANTED / EXPERIMENTAL CONTROLS UNLOCKED":"";
    cheatMenu.hidden=false;
    if(cheatsUnlocked) wallHackToggle.focus();
    else cheatCodeInput.focus();
  }
  function closeCheatMenu() {
    if(cheatMenu.hidden) return;
    cheatMenu.hidden=true;
    if(cheatMenuWasRunning && !finishPending && !ended) {
      running=true;
      last=performance.now();
      document.body.dataset.gameState="playing";
      canvas.requestPointerLock?.();
    }
    cheatMenuWasRunning=false;
  }
  function renderLevelSelect() {
    levelGrid.replaceChildren();
    levelProgress.textContent=unlockedLevelCount+" / "+levels.length+" ROOMS UNLOCKED";
    for(let i=0;i<levels.length;i++) {
      const button=document.createElement("button");
      const unlocked=i<unlockedLevelCount;
      button.type="button";
      button.className="level-button";
      button.disabled=!unlocked;
      button.dataset.levelIndex=String(i);
      const roomName=document.createElement("span");
      roomName.textContent="ROOM "+String(i+1).padStart(2,"0")+" · "+levels[i].name;
      const status=document.createElement("span");
      status.textContent=unlocked?"UNLOCKED":"LOCKED";
      button.append(roomName,status);
      levelGrid.append(button);
    }
  }
  function openLevelSelect() {
    if(finishPending || !levelSelect.hidden) return;
    levelSelectWasRunning=running;
    if(running) {
      running=false;
      mouseFiring=false;
      if(document.pointerLockElement===canvas) document.exitPointerLock();
    }
    renderLevelSelect();
    levelSelect.hidden=false;
    const firstUnlocked=levelGrid.querySelector("button:not(:disabled)");
    firstUnlocked?.focus();
  }
  function closeLevelSelect() {
    if(levelSelect.hidden) return;
    levelSelect.hidden=true;
    if(levelSelectWasRunning && !finishPending && !ended) {
      running=true;
      last=performance.now();
      document.body.dataset.gameState="playing";
      canvas.requestPointerLock?.();
    }
    levelSelectWasRunning=false;
  }
  function unlockLevel(index) {
    const nextUnlockedCount=Math.max(unlockedLevelCount,Math.min(levels.length,index+2));
    if(nextUnlockedCount===unlockedLevelCount) return;
    unlockedLevelCount=nextUnlockedCount;
    try {
      localStorage.setItem(progressStorageKey,String(unlockedLevelCount));
    } catch(error) {
      console.warn("Could not save campaign progress.",error);
    }
  }
  function startSelectedLevel(index) {
    if(index<0 || index>=unlockedLevelCount) return;
    levelSelect.hidden=true;
    levelSelectWasRunning=false;
    levelIndex=index;
    score=0;
    elapsed=0;
    ended=false;
    won=false;
    betweenLevels=false;
    overlayAction=null;
    overlay.classList.remove("start-screen");
    menuContent.hidden=false;
    tutorial.hidden=true;
    loadLevel(levelIndex);
    running=true;
    overlay.style.display="none";
    last=performance.now();
    document.body.dataset.gameState="playing";
    canvas.requestPointerLock?.();
  }

  function playSound(type) {
    if(!audioContext) return;
    const sounds={
      jump:[220,520,.18,"triangle",.12],
      fire:[180,55,.12,"sawtooth",.13],
      ar:[240,75,.08,"square",.07],
      shotgun:[120,38,.24,"sawtooth",.2],
      enemyShot:[150,65,.14,"square",.055],
      sword:[720,180,.2,"sawtooth",.1],
      punch:[120,58,.16,"triangle",.12],
      hit:[190,48,.22,"triangle",.16],
      pickup:[480,960,.2,"sine",.1],
      possess:[260,880,.42,"sawtooth",.1],
      throw:[380,90,.2,"triangle",.1],
      win:[523,1046,.7,"sine",.12],
      lose:[220,65,.65,"triangle",.12]
    };
    const [startFrequency,endFrequency,duration,waveform,volume]=sounds[type];
    const now=audioContext.currentTime;
    const oscillator=audioContext.createOscillator();
    const gain=audioContext.createGain();
    oscillator.type=waveform;
    oscillator.frequency.setValueAtTime(startFrequency,now);
    oscillator.frequency.exponentialRampToValueAtTime(endFrequency,now+duration);
    gain.gain.setValueAtTime(volume,now);
    gain.gain.exponentialRampToValueAtTime(.001,now+duration);
    oscillator.connect(gain);
    gain.connect(audioContext.destination);
    oscillator.start(now);
    oscillator.stop(now+duration);
  }

  function loadLevel(index) {
    freeCamera.active=false;
    delete document.body.dataset.cameraMode;
    blocks = levels[index].blocks;
    player.x = 0; player.y = 0; player.verticalVelocity = 0; player.z = 7.3; player.yaw = 0; player.pitch = 0;
    enemies = levels[index].enemies.map(([x,z], i) => ({
      x,z,alive:true,health:3,phase:i*2.1,gaitPhase:i*1.7,gaitAmount:0,shootTimer:.8+i*.35,
      facing:Math.atan2(player.x-x,player.z-z),weapon:i%2===0?"ar":"shotgun"
    }));
    bullets = [];
    thrownGun = null; thrownProps=[]; droppedGuns = []; shards = []; explosions = [];
    weaponPickups=[];
    props=(levels[index].props||[]).map(([kind,x,z,y],i)=>({
      kind,x,z,y,floorY:floorHeightAt(x,z),spin:(i%3)*.4,thrown:false,vx:0,vy:0,vz:0
    }));
    heldProp=null;
    replayHistory.length=0; replayFrames=[];
    replayLastSample=0; replayStartedAt=0;
    replayPlaybackTimes=[]; replayPlaybackDuration=0;
    hasGun = true;
    weaponType=secretWeaponType||"sidearm"; lastShotAt=-Infinity; lastPunchAt=-Infinity; punchSwingUntil=0; betweenLevels = false;
    swordRestore=swordCheatEnabled?{weapon:weaponType,hasGun:true}:null;
    if(swordCheatEnabled) weaponType="sword";
    replayTitle.style.display="none";
    hint.textContent="WASD MOVE  ·  SPACE JUMP  ·  MOUSE AIM  ·  CLICK FIRE / PUNCH UNARMED  ·  Q POSSESS  ·  E PICK UP  ·  P HITBOXES  ·  C ACCESS  ·  L LEVEL SELECT  ·  F FREE CAM";
    recordReplay(performance.now(),true);
    updateHud();
  }
  function toggleFreeCamera() {
    if(!freeCamera.active) {
      freeCamera.x=player.x;
      freeCamera.y=floorHeightAt(player.x,player.z)+1.62+player.y;
      freeCamera.z=player.z;
      freeCamera.yaw=player.yaw;
      freeCamera.pitch=player.pitch;
      freeCamera.active=true;
      document.body.dataset.cameraMode="free";
      hint.textContent="FREE CAMERA · MOUSE LOOK · WASD FLY · SPACE / CTRL UP / DOWN · SHIFT BOOST · F RETURN";
    } else {
      freeCamera.active=false;
      delete document.body.dataset.cameraMode;
      hint.textContent="WASD MOVE  ·  SPACE JUMP  ·  MOUSE AIM  ·  CLICK FIRE / PUNCH UNARMED  ·  Q POSSESS  ·  E PICK UP  ·  P HITBOXES  ·  C ACCESS  ·  L LEVEL SELECT  ·  F FREE CAM";
    }
  }
  function reset() {
    levelIndex = 0; score = 0; elapsed = 0; won = false; ended = false;
    loadLevel(levelIndex);
  }
  function updateHud() {
    const remaining=enemies.filter(e => e.alive).length;
    const initialCount=levels[levelIndex].enemies.length;
    levelText.textContent = "ROOM "+String(levelIndex+1).padStart(2,"0")+" · "+levels[levelIndex].name;
    hostiles.textContent = String(remaining).padStart(2,"0");
    scoreText.textContent = String(score).padStart(2,"0");
    objectiveCopy.textContent=remaining?"ELIMINATE HOSTILES":"ROOM SECURED";
    objectiveFill.style.width=((initialCount-remaining)/initialCount*100)+"%";
    document.querySelector("#weapon").textContent=weaponType==="throwingStars"?"THROWING STARS":weaponType==="candyCane"?"CANDY CANE":weaponType.toUpperCase();
    const seconds=Math.floor(elapsed);
    document.querySelector("#timer").textContent=
      String(Math.floor(seconds/60)).padStart(2,"0")+":"+String(seconds%60).padStart(2,"0");
  }
  function recordReplay(now, force=false) {
    if(!force && now-replayLastSample<replaySampleInterval) return;
    replayLastSample=now;
    replayHistory.push({
      timestamp:now,
      slowMotion:player.y===0 && !["w","a","s","d","arrowup","arrowdown","arrowleft","arrowright"].some(key=>held.has(key)),
      player:{x:player.x,y:player.y,z:player.z,yaw:player.yaw,pitch:player.pitch,weaponType,hasGun,swordSwingRemaining:Math.max(0,swordSwingUntil-now),punchSwingRemaining:Math.max(0,punchSwingUntil-now)},
      enemies:enemies.map(enemy=>({
        x:enemy.x,z:enemy.z,alive:enemy.alive,health:enemy.health,phase:enemy.phase,
        gaitPhase:enemy.gaitPhase,gaitAmount:enemy.gaitAmount,facing:enemy.facing,weapon:enemy.weapon
      })),
      props:props.map(prop=>({...prop})),
      heldProp:heldProp?{...heldProp}:null,
      bullets:bullets.map(bullet=>({...bullet})),
      thrownGun:thrownGun?{...thrownGun}:null,
      droppedGuns:droppedGuns.map(gun=>({...gun})),
      shards:shards.map(shard=>({...shard,color:[...shard.color]})),
      explosions:explosions.map(explosion=>({...explosion}))
    });
    if(replayHistory.length>replayHistoryLimit) replayHistory.shift();
  }
  function prepareReplayPlayback() {
    replayPlaybackTimes=[0];
    for(let i=1;i<replayFrames.length;i++) {
      const previous=replayFrames[i-1], current=replayFrames[i];
      const sourceDuration=Math.max(0,current.timestamp-previous.timestamp);
      const speed=previous.slowMotion?replaySlowMotionSpeed:replayNormalSpeed;
      replayPlaybackTimes.push(replayPlaybackTimes[i-1]+sourceDuration/speed);
    }
    const lastFrame=replayFrames[replayFrames.length-1];
    const finalSpeed=lastFrame?.slowMotion?replaySlowMotionSpeed:replayNormalSpeed;
    replayPlaybackDuration=(replayPlaybackTimes[replayPlaybackTimes.length-1]||0)+replaySampleInterval/finalSpeed;
  }
  function resize() {
    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const w = Math.floor(canvas.clientWidth*dpr), h = Math.floor(canvas.clientHeight*dpr);
    if (canvas.width !== w || canvas.height !== h) { canvas.width = w; canvas.height = h; gl.viewport(0,0,w,h); }
  }
  function drawBox(view, projection, x,y,z, sx,sy,sz, color, rotation=0, pitch=0) {
    const model = multiply(translate(x,y,z), multiply(rotateY(rotation), multiply(rotateX(pitch),scale(sx,sy,sz))));
    gl.bindBuffer(gl.ARRAY_BUFFER,buffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation, false, multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation, sceneColor(color));
    gl.drawArrays(gl.TRIANGLES, 0, 36);
  }
  function drawSphere(view, projection, x,y,z, sx,sy,sz, color, rotation=0) {
    const model=multiply(translate(x,y,z),multiply(rotateY(rotation),scale(sx,sy,sz)));
    gl.bindBuffer(gl.ARRAY_BUFFER,sphereBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.TRIANGLES,0,sphereVertexCount);
  }
  function drawCupBody(view,projection,x,y,z,sx,sy,sz,color,rotation=0) {
    const model=multiply(translate(x,y,z),multiply(rotateY(rotation),scale(sx,sy,sz)));
    gl.bindBuffer(gl.ARRAY_BUFFER,cupBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.TRIANGLES,0,cupVertexCount);
  }
  function drawEnemyHitbox(view,projection,x,y,z,sx,sy,sz,rotation=0,color=[1,.12,.1]) {
    const model=multiply(translate(x,y,z),multiply(rotateY(rotation),scale(sx,sy,sz)));
    gl.bindBuffer(gl.ARRAY_BUFFER,hitboxBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.LINES,0,24);
  }
  function drawEnemyOutline(view,projection,enemy,angle,bob,stride,gaitPhase,gaitAmount,baseY=0) {
    const rightX=Math.cos(angle),rightZ=-Math.sin(angle);
    const forwardX=Math.sin(angle),forwardZ=Math.cos(angle);
    const box=(ox,oy,oz,sx,sy,sz)=>{
      drawEnemyHitbox(
        view,projection,
        enemy.x+rightX*ox+forwardX*oz,oy+bob+baseY,enemy.z+rightZ*ox+forwardZ*oz,
        sx,sy,sz,angle,[1,.035,.055]
      );
    };
    const armSwing=stride*.72;
    const leftLift=Math.max(0,Math.sin(gaitPhase))*.14*gaitAmount;
    const rightLift=Math.max(0,Math.sin(gaitPhase+Math.PI))*.14*gaitAmount;
    box(0,1.17,.08,.36,.48,.25);
    box(-.37,1.12,-armSwing*.45,.13,.42,.14);
    box(.37,1.12,armSwing*.45,.13,.42,.14);
    box(-.15,.43,stride*.7,.13,.39,.15);
    box(.15,.43,-stride*.7,.13,.39,.15);
    box(0,1.79,.02,.24,.31,.23);
  }
  function drawThrowableProp(view,projection,prop,x,y,z,rotation=0) {
    if(prop.kind==="explosiveBall") {
      drawSphere(view,projection,x,y,z,.22,.22,.22,[.88,.88,.84],rotation);
      for(const [px,py,pz] of [
        [0,1,0],[.82,.35,0],[.5,-.35,.72],[-.55,.22,.69],[-.45,-.5,-.65],[.75,-.3,-.48]
      ]) {
        const c=Math.cos(rotation),s=Math.sin(rotation);
        drawSphere(view,projection,x+(c*px+s*pz)*.2,y+py*.2,z+(-s*px+c*pz)*.2,.063,.063,.063,[.055,.055,.055],rotation);
      }
      return;
    }
    const body=prop.kind==="tumbler"
      ? {width:.105,height:.2,depth:.105,shade:.13}
      : prop.kind==="cup"
        ? {width:.1,height:.1,depth:.1,shade:.2}
        : {width:.11,height:.135,depth:.095,shade:.16};
    const part=(ox,oy,oz,sx,sy,sz,color)=>{
      const c=Math.cos(rotation),s=Math.sin(rotation);
      drawBox(view,projection,x+c*ox+s*oz,y+oy,z-s*ox+c*oz,sx,sy,sz,color,rotation);
    };
    const sphere=(ox,oy,oz,sx,sy,sz,color)=>{
      const c=Math.cos(rotation),s=Math.sin(rotation);
      drawSphere(view,projection,x+c*ox+s*oz,y+oy,z-s*ox+c*oz,sx,sy,sz,color,rotation);
    };
    drawCupBody(view,projection,x,y,z,body.width,body.height,body.depth,[body.shade,body.shade,body.shade],rotation);
    sphere(0,body.height*.96,0,body.width*.96,.014,body.depth*.96,[.24,.24,.24]);
    sphere(0,body.height*.98,0,body.width*.64,.009,body.depth*.64,[.025,.025,.025]);
    if(prop.kind==="mug") {
      part(body.width*1.25,0,0,.035,.09,.04,[.11,.11,.11]);
      part(body.width*1.75,0,0,.035,.065,.04,[.11,.11,.11]);
      part(body.width*1.25,-body.height*.48,0,.035,.035,.04,[.11,.11,.11]);
      part(body.width*1.25,body.height*.48,0,.035,.035,.04,[.11,.11,.11]);
    }
    if(prop.kind==="tumbler") {
      sphere(0,body.height*.98,0,body.width*.88,.025,body.depth*.88,[.29,.29,.29]);
      part(0,body.height*1.16,0,body.width*.48,.012,body.depth*.48,[.045,.045,.045]);
    }
  }
  function drawWindowExterior(view,projection) {
    drawBox(view,projection,0,5,-24,24,16,.25,[.38,.68,.84]);
    drawSphere(view,projection,0,-1.05,-18,13,1.8,1.1,[.19,.42,.18]);
    drawSphere(view,projection,-5.7,-.45,-16.5,4.2,1.35,.8,[.27,.53,.2]);
    drawSphere(view,projection,5.8,-.4,-17,4.5,1.5,.9,[.22,.47,.17]);
    const trees=[
      [-7.5,1.4,-16,.62,1.45],
      [-5.2,1.25,-15.5,.52,1.2],
      [-2.9,1.55,-17,.7,1.6],
      [0,1.35,-16.2,.56,1.3],
      [3.15,1.55,-17,.72,1.55],
      [5.7,1.25,-15.8,.54,1.25],
      [7.6,1.5,-17,.68,1.5]
    ];
    for(const [x,y,z,radius,height] of trees) {
      drawBox(view,projection,x,y-.55,z,.11,height*.58,.11,[.25,.18,.1]);
      drawSphere(view,projection,x,y+.55,z,radius,radius*1.18,radius,[.14,.36,.13]);
      drawSphere(view,projection,x-radius*.43,y+.37,z+.04,radius*.65,radius*.68,radius*.68,[.2,.45,.16]);
      drawSphere(view,projection,x+radius*.42,y+.38,z-.03,radius*.62,radius*.72,radius*.66,[.17,.4,.14]);
    }
  }
  function drawRoomInterior(view,projection,theme) {
    const box=(x,y,z,sx,sy,sz,color)=>{
      const shade=.74+((color[0]+color[1]+color[2])/3)*.24;
      drawBox(view,projection,x,y,z,sx,sy,sz,[shade,shade,shade]);
    };
    const floorTone=theme.floor, wallTone=theme.wall, trimTone=theme.trim;
    box(0,-.18,0,10,.16,10,floorTone);
    drawWindowExterior(view,projection);

    if(theme.layout==="house" || theme.layout==="bar") {
      for(let z=-9;z<=9;z+=.72) {
        box(0,-.01,z,9.85,.008,.012,[floorTone[0]*.72,floorTone[1]*.72,floorTone[2]*.72]);
        const seamX=(Math.floor((z+9)/.72)%2)*.9-4.5;
        box(seamX,-.01,z+.34,.012,.008,.65,[floorTone[0]*.68,floorTone[1]*.68,floorTone[2]*.68]);
      }
    } else {
      const step=theme.layout==="store" || theme.layout==="station"?2:3;
      for(let line=-9;line<=9;line+=step) {
        box(0,-.01,line,9.85,.008,.018,[floorTone[0]*.72,floorTone[1]*.72,floorTone[2]*.72]);
        box(line,-.01,0,.018,.008,9.85,[floorTone[0]*.72,floorTone[1]*.72,floorTone[2]*.72]);
      }
    }

    const wallHalfHeight=theme.twoStory?3.2:1.7;
    const wallCenter=wallHalfHeight;
    const windowHalfWidth=2.7,windowBottom=1.2,windowTop=2.85;
    box(-(windowHalfWidth+10)/2,wallCenter,-9.88,(10-windowHalfWidth)/2,wallHalfHeight,.16,wallTone);
    box((windowHalfWidth+10)/2,wallCenter,-9.88,(10-windowHalfWidth)/2,wallHalfHeight,.16,wallTone);
    box(0,windowBottom/2,-9.88,windowHalfWidth,windowBottom/2,.16,wallTone);
    const upperWallHeight=wallHalfHeight*2-windowTop;
    box(0,windowTop+upperWallHeight/2,-9.88,windowHalfWidth,upperWallHeight/2,.16,wallTone);
    box(0,wallCenter,9.88,10,wallHalfHeight,.16,wallTone);
    box(-9.88,wallCenter,0,.16,wallHalfHeight,10,wallTone);
    box(9.88,wallCenter,0,.16,wallHalfHeight,10,wallTone);
    box(0,.14,-9.77,9.8,.14,.035,trimTone);
    box(0,.14,9.77,9.8,.14,.035,trimTone);
    box(-9.77,.14,0,.035,.14,9.8,trimTone);
    box(9.77,.14,0,.035,.14,9.8,trimTone);
    const windowFrame=[.7,.73,.7];
    for(const x of [-windowHalfWidth,0,windowHalfWidth]) {
      box(x,(windowBottom+windowTop)/2,-9.67,x===0?.045:.075,(windowTop-windowBottom)/2,.1,windowFrame);
    }
    box(0,windowBottom,-9.67,windowHalfWidth,.075,.1,windowFrame);
    box(0,windowTop,-9.67,windowHalfWidth,.075,.1,windowFrame);
    const ceilingY=theme.twoStory?6.48:3.43;
    box(0,ceilingY,0,10,.12,10,[.28,.3,.31]);
    box(0,ceilingY-.09,0,8,.035,8,[.34,.36,.36]);
    for(let x=-6;x<=6;x+=4) box(x,ceilingY-.14,0,.055,.025,15,[.82,.76,.58]);

    if(theme.layout==="house") {
      const fabric=[.42,.16,.13], wood=[.29,.17,.1], cloth=[.58,.27,.2];
      box(-6,.34,-5,1.65,.34,.58,fabric);
      box(-6,.78,-5.35,1.65,.52,.18,fabric);
      box(-6.86,.62,-5,.13,.35,.68,wood);
      box(-5.14,.62,-5,.13,.35,.68,wood);
      box(-6.55,.47,-4.73,.46,.17,.09,cloth);
      box(-5.45,.47,-4.73,.46,.17,.09,cloth);
      box(-2.2,.33,-4,1.05,.08,.72,wood);
      for(const x of [-2.95,-1.45]) for(const z of [-4.48,-3.52]) box(x,.15,z,.07,.2,.07,wood);
      box(-8,1.05,-9.66,1.5,.82,.16,[.16,.19,.2]);
      box(-8,1.12,-9.53,1.25,.62,.025,[.055,.1,.13]);
      box(7.7,1.15,-7.2,.85,1.15,.85,[.66,.68,.65]);
      box(5.5,.6,-8.7,2.1,.55,.42,[.38,.26,.16]);
      box(5.5,1.1,-8.65,2.1,.12,.46,[.26,.17,.11]);
      box(0,.012,0,4.8,.012,3.4,[.38,.19,.16]);
    } else if(theme.layout==="store") {
      const shelf=[.22,.27,.27], shelfEdge=[.12,.16,.17];
      for(const x of [-5,5]) {
        box(x,1,0,.1,.9,2.8,shelf);
        for(const y of [.3,.75,1.2,1.65]) {
          box(x,y,0,.38,.04,2.8,shelfEdge);
          for(let z=-2;z<=2;z+=1) {
            const hue=Math.abs(Math.sin(x*3+y*2+z));
            box(x+(z%2?-.12:.12),y+.14,z,.11,.14,.16,[.5+hue*.35,.14+hue*.2,.1+hue*.12]);
          }
        }
      }
      box(-7.2,.6,-6.7,2.6,.6,.75,[.3,.28,.2]);
      box(-7.2,1.23,-6.7,2.8,.09,.9,[.16,.21,.19]);
      box(0,2.5,-9.7,5.6,.42,.12,[.08,.24,.22]);
      box(0,2.51,-9.61,4.2,.07,.025,[.84,.77,.53]);
      box(8,.72,5.8,.75,.72,.75,[.68,.22,.12]);
      box(8,1.5,5.8,.78,.08,.8,[.85,.78,.61]);
    } else if(theme.layout==="bar") {
      const wood=[.26,.12,.075], brass=[.7,.38,.12];
      box(0,.82,-6.6,7.8,.18,.72,wood);
      box(0,.98,-6.6,7.9,.055,.76,brass);
      box(0,.48,-6.25,7.4,.48,.12,wood);
      for(let x=-3;x<=3;x+=1.5) {
        box(x,.5,-5.45,.12,.5,.12,wood);
        box(x,.78,-5.45,.46,.07,.46,brass);
        for(const z of [-7,-6.65]) box(x-.35,1.25,z,.08,.23,.08,[.2,.43,.27]);
      }
      for(const x of [-5,0,5]) {
        box(x,.58,0,.68,.1,.68,wood);
        for(const dx of [-.5,.5]) for(const dz of [-.5,.5]) box(x+dx,.29,dz,.06,.29,.06,wood);
        for(const dx of [-.26,.26]) box(x+dx,.93,0,.045,.36,.045,wood);
        box(x,.95,0,.65,.065,.65,brass);
      }
      box(0,2.05,-9.7,7.2,.72,.12,[.22,.1,.08]);
      for(let x=-3;x<=3;x+=1) box(x,2,-9.59,.42,.5,.035,[.42,.19,.08]);
      box(-7,1.2,-9.6,2.2,.35,.08,[.8,.58,.28]);
    } else if(theme.layout==="warehouse") {
      for(const z of [-8,-4,0,4,8]) {
        box(-9.6,2.1,z,.1,.08,3.7,trimTone);
        box(9.6,2.1,z,.1,.08,3.7,trimTone);
      }
      for(const x of [-6,-2,2,6]) {
        box(x,.6,-7.8,.9,.6,.9,[.42,.25,.12]);
        box(x,.6,7.6,.9,.6,.9,[.42,.25,.12]);
        box(x,1.55,7.6,.9,.35,.9,[.32,.19,.1]);
        box(x,1.23,-7.8,.9,.035,.9,trimTone);
      }
      box(0,2.25,-9.7,5.5,.75,.12,[.21,.25,.25]);
      for(let x=-2;x<=2;x+=1) box(x,1.9,-9.6,.045,.55,.03,[.47,.5,.48]);
      for(const x of [-7,7]) box(x,2.75,0,.22,.1,2.8,[.88,.75,.43]);
    } else if(theme.layout==="office") {
      for(const x of [-6,0,6]) {
        box(x,.78,-4,2.6,.12,1.15,[.38,.3,.23]);
        box(x,.95,-4.3,.95,.62,.09,[.16,.2,.22]);
        box(x,.62,4,2.6,.12,1.15,[.38,.3,.23]);
        box(x,1.02,3.7,.95,.62,.09,[.16,.2,.22]);
        for(const dx of [-1,1]) {
          box(x+dx,.38,-4.35,.07,.38,.07,[.23,.22,.2]);
          box(x+dx,.38,3.65,.07,.38,.07,[.23,.22,.2]);
        }
        box(x,1.18,-3.5,.16,.06,.16,[.62,.65,.6]);
      }
      for(const x of [-7,7]) {
        box(x,1.15,-9.68,1.55,.9,.12,[.29,.34,.34]);
        box(x,1.17,-9.58,1.25,.62,.03,[.12,.24,.3]);
      }
      box(0,2.45,-9.68,5.2,.3,.12,[.18,.32,.38]);
    } else {
      box(0,-.005,-7,8,.025,2.2,[.13,.14,.15]);
      for(let x=-3.8;x<=3.8;x+=.35) box(x,-.005,-7,.055,.025,2.2,[.65,.64,.58]);
      for(const x of [-8,-4,0,4,8]) {
        box(x,1.65,-9.3,.35,1.65,.35,[.38,.4,.4]);
        box(x,1.65,9.3,.35,1.65,.35,[.38,.4,.4]);
      }
      for(const x of [-5,0,5]) {
        box(x,.65,5,.9,.12,1.9,[.22,.25,.26]);
        for(const z of [4.3,5.7]) box(x,.35,z,.08,.35,.08,[.42,.43,.4]);
        box(x,1.1,-9.65,2.5,.68,.08,[.16,.28,.35]);
      }
      box(0,2.55,-9.62,5.8,.35,.1,[.65,.22,.15]);
      box(0,2.56,-9.54,4.5,.055,.025,[.88,.82,.65]);
    }
    if(theme.twoStory) {
      box(0,2.94,-6.625,9.8,.08,3.375,[.85,.85,.85]);
      for(let i=0;i<6;i++) {
        const top=(i+1)*.5;
        const z=2.71-i*1.08;
        box(-2.4,top/2,z,1.6,top/2,.54,[.88,.88,.88]);
        box(-4.05,top+.18,z,.055,.18,.055,[.72,.72,.72]);
        box(-.75,top+.18,z,.055,.18,.055,[.72,.72,.72]);
      }
      box(-4.05,1.75,0,.055,.08,3.15,[.72,.72,.72]);
      box(-.75,1.75,0,.055,.08,3.15,[.72,.72,.72]);
      for(const x of [-7,-4,0,4,7]) {
        if(x< -4 || x>-.8) box(x,3.4,-3.22,.7,.42,.045,[.72,.72,.72]);
      }
    }
  }
  function drawEnemyModel(view,projection,enemy,angle,bob,stride,gaitPhase,gaitAmount,baseY=0,aiming=false) {
    const rightX=Math.cos(angle), rightZ=-Math.sin(angle);
    const forwardX=Math.sin(angle), forwardZ=Math.cos(angle);
    const sphere=(ox,oy,oz,sx,sy,sz,color)=>{
      drawSphere(
        view,projection,
        enemy.x+rightX*ox+forwardX*oz,
        oy+bob+baseY,
        enemy.z+rightZ*ox+forwardZ*oz,
        sx,sy,sz,color,angle
      );
    };
    const box=(ox,oy,oz,sx,sy,sz,color)=>{
      drawBox(
        view,projection,
        enemy.x+rightX*ox+forwardX*oz,
        oy+bob+baseY,
        enemy.z+rightZ*ox+forwardZ*oz,
        sx,sy,sz,color,angle
      );
    };
    const darkRed=[.24,.018,.032], deepRed=[.39,.025,.042];
    const uniformRed=[.62,.045,.065], armorRed=[.78,.075,.09];
    const brightRed=[.96,.14,.15], trimRed=[.42,.022,.035];

    sphere(0,1.08,stride*.045,.3,.42,.2,uniformRed);
    sphere(0,1.36,stride*.035,.32,.2,.205,deepRed);
    box(0,1.16,.19+stride*.045,.38,.34,.055,armorRed);
    box(0,1.36,.205+stride*.035,.28,.09,.045,brightRed);
    box(0,1.04,.215+stride*.045,.23,.055,.035,trimRed);
    box(0,.84,.005,.3,.14,.19,deepRed);
    box(0,.84,.11,.24,.055,.035,brightRed);

    const armSwing=stride*.72;
    sphere(-.34,1.39,-armSwing*.35,.16,.17,.17,brightRed);
    sphere(.34,1.39,armSwing*.35,.16,.17,.17,brightRed);
    sphere(-.38,1.17,-armSwing*.55,.105,.25,.11,uniformRed);
    sphere(.38,aiming?1.27:1.17,armSwing*.55,.105,.25,.11,uniformRed);
    sphere(-.39-armSwing*.55,.91,.12+armSwing*.38,.085,.18,.09,armorRed);
    sphere(.39+armSwing*.55,aiming ? 1.01 : .91,.13-armSwing*.38,.085,.18,.09,armorRed);
    sphere(-.39-armSwing*.55,.78,.17+armSwing*.38,.09,.075,.12,deepRed);
    sphere(.39+armSwing*.55,aiming ? .88 : .78,.18-armSwing*.38,.09,.075,.12,deepRed);

    const leftLift=Math.max(0,Math.sin(gaitPhase))*.14*gaitAmount;
    const rightLift=Math.max(0,Math.sin(gaitPhase+Math.PI))*.14*gaitAmount;
    sphere(-.15,.62,stride,.115,.25,.12,uniformRed);
    sphere(.15,.62,-stride,.115,.25,.12,uniformRed);
    sphere(-.15,.31+leftLift,stride*1.35,.105,.2,.11,armorRed);
    sphere(.15,.31+rightLift,-stride*1.35,.105,.2,.11,armorRed);
    sphere(-.15,.105+leftLift,stride*1.35+.045,.12,.09,.17,deepRed);
    sphere(.15,.105+rightLift,-stride*1.35+.045,.12,.09,.17,deepRed);
    box(-.15,.34+leftLift,stride*1.35+.105,.11,.12,.035,brightRed);
    box(.15,.34+rightLift,-stride*1.35+.105,.11,.12,.035,brightRed);

    sphere(0,1.7,.015,.205,.235,.19,uniformRed);
    sphere(0,1.88,.005,.235,.13,.22,deepRed);
    box(0,1.79,.197,.26,.085,.035,darkRed);
    box(0,1.72,.224,.18,.045,.02,brightRed);
    sphere(-.072,1.75,.243,.023,.02,.012,brightRed);
    sphere(.072,1.75,.243,.023,.02,.012,brightRed);
    box(-.205,1.87,0,.045,.11,.07,armorRed);
    box(.205,1.87,0,.045,.11,.07,armorRed);

    box(-.11,1.23,.25-armSwing*.55,.035,.075,.025,brightRed);
    box(.11,1.23,.25+armSwing*.55,.035,.075,.025,brightRed);
    box(0,1.49,.212,.12,.035,.025,brightRed);
  }
  function drawWeaponPart(view,projection,x,y,z,offsetX,offsetY,offsetZ,sx,sy,sz,color,rotation,pitch=0) {
    const c=Math.cos(rotation),s=Math.sin(rotation),cp=Math.cos(pitch),sp=Math.sin(pitch);
    const pitchedY=cp*offsetY-sp*offsetZ;
    const pitchedZ=sp*offsetY+cp*offsetZ;
    const model=multiply(
      translate(x+c*offsetX+s*pitchedZ,y+pitchedY,z-s*offsetX+c*pitchedZ),
      multiply(rotateY(rotation),multiply(rotateX(pitch),scale(sx,sy,sz)))
    );
    gl.bindBuffer(gl.ARRAY_BUFFER,buffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.TRIANGLES,0,36);
  }
  function drawWeaponCylinder(view,projection,x,y,z,offsetX,offsetY,offsetZ,sx,sy,sz,color,rotation,pitch=0) {
    const c=Math.cos(rotation),s=Math.sin(rotation),cp=Math.cos(pitch),sp=Math.sin(pitch);
    const pitchedY=cp*offsetY-sp*offsetZ;
    const pitchedZ=sp*offsetY+cp*offsetZ;
    const model=multiply(
      translate(x+c*offsetX+s*pitchedZ,y+pitchedY,z-s*offsetX+c*pitchedZ),
      multiply(rotateY(rotation),multiply(rotateX(pitch),scale(sx,sy,sz)))
    );
    gl.bindBuffer(gl.ARRAY_BUFFER,barrelBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.TRIANGLES,0,barrelVertexCount);
  }
  function drawWeaponRoundedPart(view,projection,x,y,z,offsetX,offsetY,offsetZ,sx,sy,sz,color,rotation,pitch=0) {
    const c=Math.cos(rotation),s=Math.sin(rotation),cp=Math.cos(pitch),sp=Math.sin(pitch);
    const pitchedY=cp*offsetY-sp*offsetZ;
    const pitchedZ=sp*offsetY+cp*offsetZ;
    const model=multiply(
      translate(x+c*offsetX+s*pitchedZ,y+pitchedY,z-s*offsetX+c*pitchedZ),
      multiply(rotateY(rotation),multiply(rotateX(pitch),scale(sx,sy,sz)))
    );
    gl.bindBuffer(gl.ARRAY_BUFFER,sphereBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(color));
    gl.drawArrays(gl.TRIANGLES,0,sphereVertexCount);
  }
  function drawSwordBlade(view,projection,x,y,z,rotation,pitch=0,size=.78) {
    const model=multiply(
      translate(x,y,z),
      multiply(rotateY(rotation),multiply(rotateX(pitch),scale(size,size,size)))
    );
    gl.bindBuffer(gl.ARRAY_BUFFER,swordBladeBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor([.72,.79,.82]));
    gl.drawArrays(gl.TRIANGLES,0,swordBladeVertexCount);
  }
  function drawThrowingStar(view,projection,x,y,z,rotation=0,pitch=0,spin=0,size=1) {
    const model=multiply(translate(x,y,z),multiply(rotateY(rotation),multiply(rotateX(pitch),multiply(rotateZ(spin),scale(size,size,size)))));
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor([.68,.72,.74]));
    gl.bindBuffer(gl.ARRAY_BUFFER,throwingStarBuffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.drawArrays(gl.TRIANGLES,0,throwingStarVertexCount);
    drawSphere(view,projection,x,y,z,.045*size,.045*size,.045*size,[.32,.35,.36]);
  }
  function drawCandyCane(view,projection,x,y,z,rotation,pitch,size) {
    const points=[
      [0,.62],[0,.49],[0,.36],[0,.23],[0,.1],[0,-.03],[0,-.16],[0,-.29],[0,-.39],
      [-.07,-.47],[-.18,-.5],[-.29,-.46],[-.37,-.37],[-.38,-.26],[-.32,-.19]
    ];
    for(let i=1;i<points.length;i++) {
      const [startX,startY]=points[i-1],[endX,endY]=points[i];
      const dx=endX-startX,dy=endY-startY,length=Math.hypot(dx,dy)*size;
      const segment=multiply(rotateZ(Math.atan2(dx,-dy)),multiply(rotateX(Math.PI/2),scale(.07*size,.07*size,length)));
      const center=multiply(translate((startX+endX)*size/2,(startY+endY)*size/2,0),segment);
      const orientation=multiply(rotateZ(Math.PI/2),rotateX(-Math.PI/2));
      const model=multiply(translate(x,y,z),multiply(rotateY(rotation),
        multiply(rotateX(pitch),multiply(orientation,center))));
      gl.bindBuffer(gl.ARRAY_BUFFER,barrelBuffer);
      gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
      gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
      gl.uniform3fv(colorLocation,sceneColor(i%2===0?[.88,.035,.045]:[.98,.94,.84]));
      gl.drawArrays(gl.TRIANGLES,0,barrelVertexCount);
    }
  }
  function drawWeapon(view,projection,weapon,x,y,z,rotation=0,pitch=0,modelScale=.78) {
    if(weapon==="throwingStars") {
      drawThrowingStar(view,projection,x,y,z,rotation,pitch,0,.34);
      return;
    }
    if(weapon==="candyCane") {
      drawCandyCane(view,projection,x,y,z,rotation,pitch,modelScale*1.25);
      return;
    }
    if(weapon==="sword") {
      const swordSize=.64;
      const swordRotation=-rotation;
      const rounded=(ox,oy,oz,sx,sy,sz,color)=>drawWeaponRoundedPart(
        view,projection,x,y,z,ox*swordSize,oy*swordSize,oz*swordSize,
        sx*swordSize,sy*swordSize,sz*swordSize,color,swordRotation,pitch
      );
      const part=(ox,oy,oz,sx,sy,sz,color)=>drawWeaponPart(
        view,projection,x,y,z,ox*swordSize,oy*swordSize,oz*swordSize,
        sx*swordSize,sy*swordSize,sz*swordSize,color,swordRotation,pitch
      );
      drawSwordBlade(view,projection,x,y,z,swordRotation,pitch,.78*swordSize);
      rounded(0,.014,-.42,.009,.004,.29,[.32,.4,.44]);
      rounded(-.052,.008,-.36,.006,.003,.31,[.88,.9,.88]);
      rounded(0,0,.005,.135,.022,.035,[.62,.65,.64]);
      rounded(0,-.005,.17,.035,.035,.15,[.25,.13,.075]);
      for(const offset of [-.09,-.045,0,.045,.09]) {
        part(0,-.005,.17+offset,.038,.039,.012,[.48,.3,.16]);
      }
      rounded(0,-.005,.31,.045,.045,.045,[.48,.35,.19]);
      return;
    }
    const rifle=weapon==="ar", shotgun=weapon==="shotgun";
    const part=(ox,oy,oz,sx,sy,sz,color)=>drawWeaponPart(
      view,projection,x,y,z,ox*modelScale,oy*modelScale,oz*modelScale,
      sx*modelScale,sy*modelScale,sz*modelScale,color,rotation,pitch
    );
    const rounded=(ox,oy,oz,sx,sy,sz,color)=>drawWeaponRoundedPart(
      view,projection,x,y,z,ox*modelScale,oy*modelScale,oz*modelScale,
      sx*modelScale,sy*modelScale,sz*modelScale,color,rotation,pitch
    );
    const tube=(ox,oy,oz,sx,sy,sz,color)=>drawWeaponCylinder(
      view,projection,x,y,z,ox*modelScale,oy*modelScale,oz*modelScale,
      sx*modelScale,sy*modelScale,sz*modelScale,color,rotation,pitch
    );
    if(rifle) {
      rounded(0,0,-.04,.085,.075,.27,[.18,.2,.21]);
      rounded(0,.015,-.36,.075,.065,.2,[.24,.26,.26]);
      tube(0,.012,-.68,.035,.035,.48,[.12,.13,.14]);
      tube(0,.012,-.94,.044,.044,.055,[.24,.25,.25]);
      part(0,.012,-.975,.05,.05,.018,[.045,.05,.05]);
      rounded(0,-.015,.37,.075,.078,.28,[.16,.18,.19]);
      rounded(0,-.19,.025,.056,.16,.075,[.19,.2,.19]);
      part(0,-.055,.005,.09,.08,.12,[.16,.17,.16]);
      part(0,-.12,-.025,.018,.1,.1,[.075,.08,.075]);
      rounded(0,.07,.005,.055,.11,.08,[.17,.18,.18]);
      part(0,.135,-.025,.045,.025,.19,[.12,.13,.14]);
      part(0,.145,-.38,.025,.04,.04,[.09,.1,.1]);
      part(0,.12,.25,.025,.035,.04,[.09,.1,.1]);
      part(.078,.02,-.04,.012,.045,.13,[.3,.32,.31]);
      part(-.078,.02,-.04,.012,.045,.13,[.3,.32,.31]);
      part(0,.08,-.13,.14,.018,.25,[.11,.12,.12]);
      part(0,.105,-.2,.025,.03,.045,[.07,.08,.08]);
      return;
    }
    if(shotgun) {
      rounded(0,0,.035,.09,.075,.24,[.22,.19,.16]);
      rounded(0,.015,-.25,.06,.06,.2,[.26,.22,.18]);
      tube(-.035,.015,-.62,.028,.028,.62,[.13,.14,.15]);
      tube(.035,.015,-.62,.028,.028,.62,[.13,.14,.15]);
      tube(-.035,.015,-.94,.035,.035,.045,[.24,.25,.25]);
      tube(.035,.015,-.94,.035,.035,.045,[.24,.25,.25]);
      rounded(0,-.01,.36,.08,.085,.3,[.34,.2,.11]);
      rounded(0,-.16,.08,.07,.15,.08,[.25,.17,.12]);
      tube(0,-.1,-.27,.035,.045,.28,[.15,.16,.16]);
      rounded(0,.075,-.12,.05,.045,.1,[.12,.13,.13]);
      part(0,.12,-.27,.025,.04,.045,[.09,.1,.1]);
      part(0,-.02,-.63,.085,.018,.14,[.3,.31,.3]);
      part(0,-.045,-.36,.16,.018,.2,[.12,.13,.13]);
      return;
    }
    rounded(0,0,-.035,.07,.06,.22,[.19,.2,.21]);
    rounded(0,.04,-.055,.067,.04,.2,[.3,.31,.3]);
    tube(0,.015,-.25,.026,.026,.2,[.12,.13,.14]);
    tube(0,.015,-.36,.035,.035,.035,[.22,.23,.23]);
    part(0,-.1,.075,.06,.085,.055,[.23,.18,.13]);
    rounded(0,-.145,.105,.055,.105,.065,[.25,.19,.14]);
    rounded(0,-.07,.13,.065,.035,.035,[.3,.31,.3]);
    part(0,-.105,.12,.015,.11,.075,[.17,.13,.1]);
    part(0,.085,-.1,.02,.025,.035,[.11,.12,.12]);
    part(0,.09,.09,.02,.025,.035,[.11,.12,.12]);
    part(.055,.005,-.015,.012,.035,.075,[.38,.39,.36]);
    part(-.055,.005,-.015,.012,.035,.075,[.38,.39,.36]);
    part(0,-.025,-.125,.105,.012,.045,[.1,.11,.11]);
  }
  function drawTrail(view, projection, bullet) {
    if(bullet.weapon==="candyCane") {
      const speed=Math.hypot(bullet.vx,bullet.vy,bullet.vz)||1;
      const directionX=bullet.vx/speed,directionY=bullet.vy/speed,directionZ=bullet.vz/speed;
      const yaw=Math.atan2(-directionX,-directionZ);
      const pitch=-Math.asin(Math.max(-1,Math.min(1,directionY)));
      const size=.4,tipOffset=.62*size;
      drawCandyCane(view,projection,
        bullet.x-directionX*tipOffset,bullet.y-directionY*tipOffset,bullet.z-directionZ*tipOffset,
        yaw,pitch,size);
      return;
    }
    if(bullet.weapon==="throwingStars") {
      const yaw=Math.atan2(bullet.vx,bullet.vz);
      const pitch=Math.atan2(bullet.vy,Math.hypot(bullet.vx,bullet.vz));
      drawThrowingStar(view,projection,bullet.x,bullet.y,bullet.z,yaw,pitch,bullet.spin||0,.72);
      return;
    }
    const length = .72;
    const speed = Math.hypot(bullet.vx,bullet.vy,bullet.vz) || 1;
    const dx=bullet.vx/speed, dy=bullet.vy/speed, dz=bullet.vz/speed;
    const model = multiply(translate(bullet.x-dx*length*.18,bullet.y-dy*length*.18,bullet.z-dz*length*.18),
      multiply(rotateY(Math.atan2(dx,dz)),multiply(rotateX(Math.atan2(dy,Math.hypot(dx,dz))),scale(.025,.025,length/2))));
    gl.uniformMatrix4fv(mvpLocation,false,multiply(projection,multiply(view,model)));
    gl.uniform3fv(colorLocation,sceneColor(bullet.team==="hostile"?[1,.12,.08]:[1,.78,.38]));
    gl.bindBuffer(gl.ARRAY_BUFFER,buffer);
    gl.vertexAttribPointer(position,3,gl.FLOAT,false,0,0);
    gl.drawArrays(gl.TRIANGLES,0,36);
  }
  function fireEnemyShot(enemy, angle) {
    playSound("enemyShot");
    const rightX=Math.cos(angle), rightZ=-Math.sin(angle);
    const forwardX=Math.sin(angle), forwardZ=Math.cos(angle);
    const x=enemy.x+rightX*.38+forwardX*.86;
    const y=floorHeightAt(enemy.x,enemy.z)+.9+Math.sin(enemy.phase)*.025;
    const z=enemy.z+rightZ*.38+forwardZ*.86;
    const dx=player.x-x, dy=floorHeightAt(player.x,player.z)+1.25+player.y-y, dz=player.z-z;
    const distance=Math.hypot(dx,dy,dz)||1;
    const speed=8.5;
    bullets.push({x,y,z,vx:dx/distance*speed,vy:dy/distance*speed,vz:dz/distance*speed,life:2.8,team:"hostile"});
  }
  function floorHeightAt(x,z) {
    if(!levels[levelIndex]?.twoStory) return 0;
    if(z<-3.25) return 3;
    if(x>=-4 && x<=-.8 && z<=3.25) {
      return Math.min(3,Math.max(0,Math.ceil((3.25-z)/1.08)*.5));
    }
    return 0;
  }
  function enemyHitboxContains(enemy,x,y,z) {
    const floorY=floorHeightAt(enemy.x,enemy.z);
    const bob=Math.sin(enemy.phase)*.022+Math.sin(enemy.phase*.47)*.009;
    return Math.abs(x-enemy.x)<=enemyHitboxHalfWidth
      && Math.abs(z-enemy.z)<=enemyHitboxHalfDepth
      && Math.abs(y-(floorY+enemyHitboxHalfHeight+bob))<=enemyHitboxHalfHeight;
  }
  function intersectsBlock(x,z,r=.32,height=floorHeightAt(x,z)) {
    if (Math.abs(x)>9.35 || Math.abs(z)>9.35) return true;
    if(levels[levelIndex]?.twoStory && z<-3.25 && z> -3.65 && (x<-4 || x>-.8)) return true;
    const activeBlocks=height>=2.5?levels[levelIndex].upperBlocks||[]:blocks;
    return activeBlocks.some(b => x > b.x-b.w/2-r && x < b.x+b.w/2+r && z > b.z-b.d/2-r && z < b.z+b.d/2+r);
  }
  function enemyCanSeePlayer(enemy,target=player) {
    const dx=target.x-enemy.x,dz=target.z-enemy.z;
    const distance=Math.hypot(dx,dz);
    if(distance<.001 || distance>13) return false;
    const enemyFloor=floorHeightAt(enemy.x,enemy.z),playerFloor=floorHeightAt(target.x,target.z);
    if(Math.abs(enemyFloor-playerFloor)>1.25 || target.y>.65) return false;
    const facing=enemy.facing??Math.atan2(dx,dz);
    const facingDot=(dx*Math.sin(facing)+dz*Math.cos(facing))/distance;
    if(facingDot<.45) return false;
    const samples=Math.ceil(distance/.18);
    for(let i=1;i<samples;i++) {
      const along=i/samples;
      const x=enemy.x+dx*along,z=enemy.z+dz*along;
      if(intersectsBlock(x,z,.025,floorHeightAt(x,z))) return false;
    }
    return true;
  }
  function aimedEnemy() {
    const eyeY=floorHeightAt(player.x,player.z)+1.62+player.y;
    const dx=Math.sin(player.yaw)*Math.cos(player.pitch);
    const dy=Math.sin(player.pitch);
    const dz=-Math.cos(player.yaw)*Math.cos(player.pitch);
    let nearest=null, nearestAlong=Infinity;
    for(const enemy of enemies) {
      if(!enemy.alive) continue;
      const vx=enemy.x-player.x, vy=floorHeightAt(enemy.x,enemy.z)+1.15-eyeY, vz=enemy.z-player.z;
      const along=vx*dx+vy*dy+vz*dz;
      if(along<=0 || along>18 || along>=nearestAlong) continue;
      const miss=Math.hypot(vx-dx*along,vy-dy*along,vz-dz*along);
      if(miss>.48) continue;
      let blocked=false;
      const samples=Math.ceil(along/.12);
      for(let i=1;i<samples-2;i++) {
        const distance=along*i/samples;
        const x=player.x+dx*distance,z=player.z+dz*distance;
        if(intersectsBlock(x,z,.02,floorHeightAt(x,z))) { blocked=true; break; }
      }
      if(!blocked) { nearest=enemy; nearestAlong=along; }
    }
    return nearest;
  }
  function possessAimedEnemy() {
    if(!running) return;
    const now=performance.now();
    const remaining=5000-(now-lastPossessAt);
    if(remaining>0) {
      toast.textContent="POSSESS READY IN "+(remaining/1000).toFixed(1)+"s";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},650);
      return;
    }
    const target=aimedEnemy();
    if(!target) {
      toast.textContent="AIM AT AN OPERATIVE";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},650);
      return;
    }
    lastPossessAt=now;
    playSound("possess");
    const oldX=player.x, oldZ=player.z;
    const oldFloor=floorHeightAt(oldX,oldZ);
    explode(oldX,oldFloor+1.05,oldZ);
    for(let i=0;i<10;i++) {
      const direction=Math.random()*Math.PI*2, speed=1+Math.random()*2.5;
      shards.push({
        x:oldX,y:oldFloor+.45+Math.random()*1.1,z:oldZ,floorY:oldFloor,
        vx:Math.cos(direction)*speed,vy:1+Math.random()*2,vz:Math.sin(direction)*speed,
        size:.05+Math.random()*.06,life:.5+Math.random()*.4,spin:Math.random()*Math.PI*2,
        color:[.8,.78,.72]
      });
    }
    defeatEnemy(target);
    player.x=target.x;
    player.y=0;
    player.verticalVelocity=0;
    player.z=target.z;
    hasGun=true;
    weaponType=target.weapon;
    updateHud();
    if(!enemies.some(enemy=>enemy.alive)) finish(true);
    toast.textContent="POSSESSED";
    toast.classList.add("on");
    clearTimeout(toastTimer);
    toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},800);
  }
  function explode(x,y,z) {
    explosions.push({x,y,z,life:.42,maxLife:.42});
    const floorY=floorHeightAt(x,z);
    const colors=[[1,.72,.16],[1,.3,.06],[.94,.08,.035],[1,.9,.52]];
    for(let i=0;i<24;i++) {
      const direction=Math.random()*Math.PI*2;
      const speed=1.5+Math.random()*5;
      shards.push({
        x,y,z,floorY,
        vx:Math.cos(direction)*speed,
        vy:.5+Math.random()*4.5,
        vz:Math.sin(direction)*speed,
        size:.045+Math.random()*.09,
        life:.35+Math.random()*.5,
        spin:Math.random()*Math.PI*2,
        color:colors[Math.floor(Math.random()*colors.length)]
      });
    }
  }
  function explodeCandyCane(bullet) {
    explode(bullet.x,bullet.y,bullet.z);
    playSound("hit");
    const floorY=floorHeightAt(bullet.x,bullet.z);
    const blastTargets=enemies.filter(enemy=>enemy.alive
      && Math.abs(floorY-floorHeightAt(enemy.x,enemy.z))<1.25
      && Math.hypot(bullet.x-enemy.x,bullet.z-enemy.z)<2.4);
    for(const enemy of blastTargets) defeatEnemy(enemy);
    if(blastTargets.length && !enemies.some(enemy=>enemy.alive)) finish(true);
  }
  function spawnExplosiveBall() {
    if(heldProp?.kind==="explosiveBall" || props.some(prop=>prop.kind==="explosiveBall")) {
      cheatMessage.textContent="SOCCER BALL ALREADY DEPLOYED";
      return;
    }
    let position=null;
    for(let i=0;i<16;i++) {
      const angle=player.yaw+i*Math.PI/8;
      const x=player.x+Math.sin(angle)*1.45;
      const z=player.z-Math.cos(angle)*1.45;
      if(!intersectsBlock(x,z,.24)) {
        position={x,z};
        break;
      }
    }
    if(!position) {
      cheatMessage.textContent="NO ROOM TO DEPLOY THE SOCCER BALL";
      return;
    }
    props.push({
      kind:"explosiveBall",x:position.x,y:floorHeightAt(position.x,position.z)+.23,z:position.z,floorY:floorHeightAt(position.x,position.z),
      vx:0,vy:0,vz:0,spin:0,thrown:false
    });
    playSound("pickup");
    cheatMessage.textContent="SOCCER BALL DEPLOYED / KICK OR PICK UP AND THROW";
  }
  function defeatEnemy(enemy) {
    if (!enemy.alive) return;
    playSound("hit");
    enemy.alive=false;
    score++;
    const enemyFloor=floorHeightAt(enemy.x,enemy.z);
    explode(enemy.x,enemyFloor+1.05,enemy.z);
    const angle=Math.atan2(player.x-enemy.x,player.z-enemy.z);
    droppedGuns.push({
      x:enemy.x+Math.sin(angle)*.34,y:enemyFloor+1.08,z:enemy.z+Math.cos(angle)*.34,floorY:enemyFloor,
      vx:-Math.sin(angle)*1.5+(Math.random()-.5),vy:2+Math.random(),vz:-Math.cos(angle)*1.5+(Math.random()-.5),
      spin:angle,life:18,weapon:enemy.weapon
    });
    for(let i=0;i<12;i++) {
      const direction=Math.random()*Math.PI*2, speed=1.2+Math.random()*3;
      shards.push({
        x:enemy.x+(Math.random()-.5)*.3,y:enemyFloor+.55+Math.random()*1.25,z:enemy.z+(Math.random()-.5)*.3,
        floorY:enemyFloor,
        vx:Math.cos(direction)*speed,vy:1+Math.random()*3,vz:Math.sin(direction)*speed,
        size:.045+Math.random()*.065,life:.65+Math.random()*.5,spin:Math.random()*Math.PI*2,
        color:Math.random()<.5?[.9,.07,.08]:[.58,.08,.09]
      });
    }
    updateHud();
  }
  function punchEnemy(enemy) {
    if(!enemy.alive) return false;
    enemy.health=Math.max(0,enemy.health-(onePunchEnabled?enemy.health:1));
    if(enemy.health===0) {
      defeatEnemy(enemy);
      return true;
    }
    playSound("hit");
    const facing=Math.atan2(player.x-enemy.x,player.z-enemy.z);
    for(let i=0;i<5;i++) {
      const direction=facing+(Math.random()-.5)*1.15;
      shards.push({
        x:enemy.x+(Math.random()-.5)*.24,y:floorHeightAt(enemy.x,enemy.z)+.75+Math.random()*.65,z:enemy.z+(Math.random()-.5)*.24,
        floorY:floorHeightAt(enemy.x,enemy.z),
        vx:Math.sin(direction)*(1+Math.random()*1.8),vy:.5+Math.random()*1.8,vz:Math.cos(direction)*(1+Math.random()*1.8),
        size:.035+Math.random()*.035,life:.25+Math.random()*.2,spin:Math.random()*Math.PI*2,
        color:[.9,.07,.08]
      });
    }
    return false;
  }
  function fire() {
    if (!running) return;
    const now=performance.now();
    if(!hasGun) {
      if(now-lastPunchAt<480) return;
      lastPunchAt=now;
      punchSwingUntil=now+380;
      playSound("punch");
      const target=enemies
        .filter(enemy=>enemy.alive)
        .map(enemy=>{
          const dx=enemy.x-player.x,dz=enemy.z-player.z,distance=Math.hypot(dx,dz);
          const facing=(dx*Math.sin(player.yaw)-dz*Math.cos(player.yaw))/(distance||1);
          let blocked=Math.abs(floorHeightAt(enemy.x,enemy.z)-floorHeightAt(player.x,player.z))>1.25;
          const samples=Math.ceil(distance/.12);
          for(let i=1;i<samples;i++) {
            const along=i/samples;
            if(intersectsBlock(player.x+dx*along,player.z+dz*along,.02)) {
              blocked=true;
              break;
            }
          }
          return {enemy,distance,facing,blocked};
        })
        .filter(hit=>hit.distance<1.6 && hit.facing>.45 && !hit.blocked)
        .sort((a,b)=>a.distance-b.distance)[0];
      if(target) {
        const defeated=punchEnemy(target.enemy);
        if(!enemies.some(enemy=>enemy.alive)) finish(true);
        toast.textContent=defeated?"PUNCH KNOCKOUT":"PUNCH CONNECTED · "+target.enemy.health+" HITS LEFT";
      } else {
        toast.textContent="PUNCH";
      }
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},260);
      return;
    }
    const cooldown=weaponType==="candyCane"?850:weaponType==="throwingStars"?320:(noReloadEnabled?0:2000);
    if(cooldown>0 && now-lastShotAt<cooldown) {
      toast.textContent="RELOADING";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},350);
      return;
    }
    lastShotAt=now;
    if(weaponType==="sword") {
      playSound("sword");
      const target=enemies
        .filter(e=>e.alive)
        .map(e=>{
          const dx=e.x-player.x,dz=e.z-player.z,distance=Math.hypot(dx,dz);
          const facing=(dx*Math.sin(player.yaw)-dz*Math.cos(player.yaw))/(distance||1);
          return {enemy:e,distance,facing};
        })
        .filter(hit=>hit.distance<3 && hit.facing>-.05)
        .sort((a,b)=>a.distance-b.distance)[0];
      swordSwingUntil=now+swordSwingDuration;
      if(target) {
        defeatEnemy(target.enemy);
        if(!enemies.some(e=>e.alive)) finish(true);
      }
      toast.textContent="SWORD SWING";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},250);
      return;
    }
    const dx = Math.sin(player.yaw)*Math.cos(player.pitch);
    const dy = Math.sin(player.pitch);
    const dz = -Math.cos(player.yaw)*Math.cos(player.pitch);
    let x,y,z;
    if(weaponType==="candyCane") {
      const weaponPitch=player.pitch, modelScale=.78*1.25;
      const sideOffset=.28, forwardOffset=.9*Math.cos(weaponPitch);
      const rootX=player.x+Math.cos(player.yaw)*sideOffset+Math.sin(player.yaw)*forwardOffset;
      const rootY=floorHeightAt(player.x,player.z)+1.55+player.y+.45*Math.sin(weaponPitch);
      const rootZ=player.z+Math.sin(player.yaw)*sideOffset-Math.cos(player.yaw)*forwardOffset;
      const muzzleOffset=.62*modelScale;
      x=rootX+dx*muzzleOffset;
      y=rootY+dy*muzzleOffset;
      z=rootZ+dz*muzzleOffset;
    } else {
      const muzzleX=player.x+Math.cos(player.yaw)*.35+Math.sin(player.yaw)*.45;
      const muzzleZ=player.z+Math.sin(player.yaw)*.35-Math.cos(player.yaw)*.45;
      x=muzzleX+dx*.32;
      y=floorHeightAt(player.x,player.z)+1.22+player.y+dy*.32;
      z=muzzleZ+dz*.32;
    }
    const speed=weaponType==="candyCane"?15:weaponType==="throwingStars"?25:weaponType==="ar"?30:weaponType==="shotgun"?19:22;
    playSound(weaponType==="throwingStars"?"throw":weaponType==="ar"?"ar":weaponType==="shotgun"?"shotgun":"fire");
    const spread=weaponType==="shotgun"?[-.12,-.08,-.04,0,.04,.08,.12]:[0];
    for(const offset of spread) {
      const shotYaw=player.yaw+offset;
      const shotPitch=player.pitch+offset*.3;
      const shotX=Math.sin(shotYaw)*Math.cos(shotPitch);
      const shotY=Math.sin(shotPitch);
      const shotZ=-Math.cos(shotYaw)*Math.cos(shotPitch);
      bullets.push({x,y,z,vx:shotX*speed,vy:shotY*speed,vz:shotZ*speed,life:weaponType==="candyCane"?3.2:weaponType==="throwingStars"?2.2:1.8,team:"player",weapon:weaponType,spin:0});
    }
    toast.classList.add("on");
    clearTimeout(toastTimer);
    toastTimer=setTimeout(()=>toast.classList.remove("on"),220);
  }
  function throwHeldItem() {
    if (!running) return;
    if(heldProp) {
      playSound("throw");
      const directionX=Math.sin(player.yaw)*Math.cos(player.pitch);
      const directionY=Math.sin(player.pitch);
      const directionZ=-Math.cos(player.yaw)*Math.cos(player.pitch);
      props.push({
        kind:heldProp.kind,
        x:player.x+Math.sin(player.yaw)*.48,
        y:floorHeightAt(player.x,player.z)+1.2+player.y,
        z:player.z-Math.cos(player.yaw)*.48,
        floorY:floorHeightAt(player.x,player.z),
        vx:directionX*12,vy:directionY*12+1.4,vz:directionZ*12,
        spin:0,thrown:true
      });
      const thrownKind=heldProp.kind==="explosiveBall"?"SOCCER BALL":heldProp.kind.toUpperCase();
      heldProp=null;
      toast.textContent=thrownKind+" THROWN";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},650);
      return;
    }
    if(weaponType==="throwingStars") {
      fire();
      return;
    }
    if(weaponType==="candyCane") {
      fire();
      return;
    }
    if(!hasGun) return;
    playSound("throw");
    const dx=Math.sin(player.yaw)*Math.cos(player.pitch);
    const dy=Math.sin(player.pitch);
    const dz=-Math.cos(player.yaw)*Math.cos(player.pitch);
    const x=player.x+Math.sin(player.yaw)*.45;
    const y=floorHeightAt(player.x,player.z)+1.28, z=player.z-Math.cos(player.yaw)*.45;
    const speed=12;
    thrownGun={x,y,z,vx:dx*speed,vy:dy*speed+1.6,vz:dz*speed,life:2.5,spin:0,weapon:weaponType,floorY:floorHeightAt(player.x,player.z)};
    hasGun=false;
    toast.textContent="GUN THROWN — RETRIEVE IT";
    toast.classList.add("on");
    clearTimeout(toastTimer);
    toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},900);
  }
  function pickUp() {
    if (!running) return;
    if(!heldProp) {
      let nearestProp=-1, nearestPropDistance=1.55;
      for(let i=0;i<props.length;i++) {
        const prop=props[i];
        if(Math.abs((prop.floorY||0)-floorHeightAt(player.x,player.z))>1.25) continue;
        const distance=Math.hypot(prop.x-player.x,prop.z-player.z);
        if(distance<nearestPropDistance) { nearestPropDistance=distance; nearestProp=i; }
      }
      if(nearestProp>=0) {
        const [prop]=props.splice(nearestProp,1);
        heldProp={kind:prop.kind};
        playSound("pickup");
        toast.textContent=(prop.kind==="explosiveBall"?"SOCCER BALL":prop.kind.toUpperCase())+" PICKED UP";
        toast.classList.add("on");
        clearTimeout(toastTimer);
        toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},650);
        return;
      }
    }
    if (!hasGun && thrownGun && Math.abs((thrownGun.floorY||0)-floorHeightAt(player.x,player.z))<1.25 && Math.hypot(thrownGun.x-player.x,thrownGun.z-player.z)<1.6) {
      playSound("pickup");
      weaponType=thrownGun.weapon||"sidearm";
      hasGun=true; thrownGun=null;
      updateHud();
      toast.textContent="GUN RECOVERED";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},700);
      return;
    }
    let pickupIndex=-1, pickupDistance=1.6;
    for(let i=0;i<weaponPickups.length;i++) {
      if(Math.abs(floorHeightAt(weaponPickups[i].x,weaponPickups[i].z)-floorHeightAt(player.x,player.z))>1.25) continue;
      const distance=Math.hypot(weaponPickups[i].x-player.x,weaponPickups[i].z-player.z);
      if(distance<pickupDistance) { pickupDistance=distance; pickupIndex=i; }
    }
    if(pickupIndex>=0) {
      playSound("pickup");
      const [pickup]=weaponPickups.splice(pickupIndex,1);
      weaponType=pickup.weapon; hasGun=true;
      updateHud();
      toast.textContent=weaponType.toUpperCase()+" PICKED UP";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},700);
      return;
    }
    let nearestIndex=-1, nearestDistance=1.6;
    for(let i=0;i<droppedGuns.length;i++) {
      const distance=Math.hypot(droppedGuns[i].x-player.x,droppedGuns[i].z-player.z);
      if(distance<nearestDistance) { nearestDistance=distance; nearestIndex=i; }
    }
    if(nearestIndex<0) return;
    const [gun]=droppedGuns.splice(nearestIndex,1);
    playSound("pickup");
    weaponType=gun.weapon||"ar";
    hasGun=true;
    updateHud();
    toast.textContent=weaponType.toUpperCase()+" PICKED UP";
    toast.classList.add("on");
    clearTimeout(toastTimer);
    toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},700);
  }
  function finish(victory) {
    if (ended || finishPending) return;
    if(victory) {
      unlockLevel(levelIndex);
      playSound("win");
      recordReplay(performance.now(),true);
      replayFrames=replayHistory.slice();
      prepareReplayPlayback();
      replayStartedAt=performance.now();
      finishPending=true;
      running=false;
      finishDeadline=replayStartedAt+replayPlaybackDuration;
      replayTitle.style.display="block";
      hint.textContent="Z  SKIP REPLAY";
      overlay.style.display="none";
      if(document.pointerLockElement===canvas) document.exitPointerLock();
      return;
    }
    showFinish(false);
  }
  function showFinish(victory) {
    if (ended) return;
    if(!victory) playSound("lose");
    replayTitle.style.display="none";
    hint.textContent="WASD MOVE  ·  SPACE JUMP  ·  MOUSE AIM  ·  CLICK FIRE / PUNCH UNARMED  ·  Q POSSESS  ·  E PICK UP  ·  P HITBOXES  ·  C ACCESS  ·  L LEVEL SELECT  ·  F FREE CAM";
    if (victory && levelIndex<levels.length-1) {
      running=false; betweenLevels=true;
      showOverlayMessage(
        "ROOM "+String(levelIndex+1).padStart(2,"0")+" SECURED",
        "NEXT ROOM",
        "The room is clear. A new floor plan and more security operatives are waiting.",
        "ENTER LEVEL "+String(levelIndex+2).padStart(2,"0"),
        begin
      );
      if (document.pointerLockElement===canvas) document.exitPointerLock();
      return;
    }
    running=false; won=victory; ended=true;
    showOverlayMessage(
      victory ? "ALL ROOMS SECURED" : "YOU WERE OVERRUN",
      victory ? "ALL CLEAR" : "GAME OVER",
      victory
        ? "Every last one is down. You made it out with your pulse still steady."
        : "The room caught up with you. Take a breath, then try the run again.",
      "PLAY AGAIN",
      begin
    );
    if (document.pointerLockElement===canvas) document.exitPointerLock();
  }
  function begin() {
    const AudioContextClass=window.AudioContext||window.webkitAudioContext;
    if(AudioContextClass) {
      if(!audioContext) audioContext=new AudioContextClass();
      if(audioContext.state==="suspended") audioContext.resume();
    }
    replayTitle.style.display="none";
    overlayAction=null;
    overlay.classList.remove("start-screen");
    menuContent.hidden=false;
    tutorial.hidden=true;
    menuContent.hidden=false;
    tutorial.hidden=true;
    if (ended) reset();
    else if (betweenLevels) {
      levelIndex++;
      loadLevel(levelIndex);
    } else if (!running && enemies.length===0) reset();
    running=true; overlay.style.display="none"; last=performance.now();
    document.body.dataset.gameState="playing";
    canvas.requestPointerLock?.();
  }
  cheatCodeForm.addEventListener("submit",e=>{
    e.preventDefault();
    if(cheatCodeInput.value.trim().toLowerCase()==="workout16") {
      onePunchEnabled=true;
      cheatMessage.textContent="ONE-PUNCH MODE ENABLED";
      cheatCodeInput.value="";
      return;
    }
    if(cheatCodeInput.value.trim().toLowerCase()==="explodedyball10") {
      spawnExplosiveBall();
      cheatCodeInput.value="";
      return;
    }
    if(cheatCodeInput.value.trim().toLowerCase()==="ninjaaa1026") {
      throwingStarsUnlocked=true;
      secretWeaponType="throwingStars";
      weaponType="throwingStars";
      hasGun=true;
      lastShotAt=-Infinity;
      updateHud();
      cheatMessage.textContent="NINJA MODE ENABLED / THROWING STARS EQUIPPED";
      cheatCodeInput.value="";
      return;
    }
    if(cheatCodeInput.value.trim().toLowerCase()==="candycanerpg1") {
      secretWeaponType="candyCane";
      weaponType="candyCane";
      hasGun=true;
      lastShotAt=-Infinity;
      updateHud();
      cheatMessage.textContent="CANDY CANE RPG EQUIPPED / EXPLOSIVE CANDY CANES READY";
      cheatCodeInput.value="";
      return;
    }
    if(cheatCodeInput.value.trim()!=="12302014") {
      cheatMessage.textContent="ACCESS DENIED / INVALID CODE";
      cheatCodeInput.select();
      return;
    }
    cheatsUnlocked=true;
    cheatOptions.hidden=false;
    wallHackToggle.checked=true;
    swordToggle.checked=true;
    noReloadToggle.checked=true;
    wallHackEnabled=true;
    noReloadEnabled=true;
    setSwordCheat(true);
    cheatMessage.textContent="ACCESS GRANTED / ALL EXPERIMENTAL CONTROLS ENABLED";
  });
  document.querySelector("#cheat-close").addEventListener("click",closeCheatMenu);
  wallHackToggle.addEventListener("change",()=>{wallHackEnabled=wallHackToggle.checked;});
  swordToggle.addEventListener("change",()=>setSwordCheat(swordToggle.checked));
  noReloadToggle.addEventListener("change",()=>{noReloadEnabled=noReloadToggle.checked;});
  sensitivitySetting.addEventListener("input",()=>{
    mouseSensitivity=Number(sensitivitySetting.value);
    sensitivityValue.value=Math.round(mouseSensitivity*100)+"%";
    try {
      localStorage.setItem("still-red-mouse-sensitivity",String(mouseSensitivity));
    } catch(error) {
      console.warn("Could not save mouse sensitivity.",error);
    }
  });
  paletteToggle.addEventListener("change",()=>{
    swappedPalette=paletteToggle.checked;
    try {
      localStorage.setItem("still-red-swapped-palette",String(swappedPalette));
    } catch(error) {
      console.warn("Could not save the color scheme setting.",error);
    }
  });
  settingsClose.addEventListener("click",closeSettings);
  levelGrid.addEventListener("click",e=>{
    const button=e.target.closest("button[data-level-index]");
    if(!button || button.disabled) return;
    startSelectedLevel(Number(button.dataset.levelIndex));
  });
  document.querySelector("#level-select-close").addEventListener("click",closeLevelSelect);
  startButton.addEventListener("click", begin);
  document.querySelector("#tutorial-open").addEventListener("click",()=>{
    showTutorialPage(0);
    overlay.classList.remove("start-screen");
    menuContent.hidden=true;
    tutorial.hidden=false;
  });
  document.querySelector("#tutorial-back").addEventListener("click",()=>{
    if(overlayAction) return;
    if(tutorialIndex===0) {
      tutorial.hidden=true;
      menuContent.hidden=false;
      overlay.classList.add("start-screen");
      return;
    }
    showTutorialPage(tutorialIndex-1);
  });
  document.querySelector("#tutorial-next").addEventListener("click",()=>{
    if(overlayAction) {
      const action=overlayAction;
      overlayAction=null;
      action();
      return;
    }
    if(tutorialIndex===tutorialPages.length-1) {
      begin();
      return;
    }
    showTutorialPage(tutorialIndex+1);
  });
  document.addEventListener("keydown", e => {
    const k=e.key.toLowerCase();
    if(!settingsMenu.hidden) {
      if(!e.repeat && k==="escape") {
        e.preventDefault();
        closeSettings();
      }
      return;
    }
    if(!cheatMenu.hidden) {
      if(!e.repeat && k==="escape") {
        e.preventDefault();
        closeCheatMenu();
      }
      return;
    }
    if(!e.repeat && k==="escape" && !finishPending && !ended) {
      e.preventDefault();
      openSettings();
      return;
    }
    if(!levelSelect.hidden) {
      if(!e.repeat && (k==="escape" || k==="l")) {
        e.preventDefault();
        closeLevelSelect();
      }
      return;
    }
    if(!e.repeat && k==="c" && !finishPending && !(e.target instanceof Element && e.target.closest("#cheat-menu"))) {
      e.preventDefault();
      openCheatMenu();
      return;
    }
    if(!e.repeat && k==="l" && !finishPending && !(e.target instanceof Element && e.target.matches("input,textarea,select"))) {
      e.preventDefault();
      openLevelSelect();
      return;
    }
  });
  canvas.addEventListener("click", () => { if (running && document.pointerLockElement!==canvas) canvas.requestPointerLock?.(); });
  document.addEventListener("pointerlockchange", () => {
    if (document.pointerLockElement!==canvas && running) {
      running=false; overlay.style.display="grid";
      document.body.dataset.gameState="menu";
      showOverlayMessage("PAUSED","STILL / RED","The room is waiting. Step back in when you are ready.","RESUME",begin);
    }
  });
  document.addEventListener("mousemove", e => {
    if (document.pointerLockElement!==canvas || !running) return;
    if(freeCamera.active) {
      freeCamera.yaw+=e.movementX*.0024*mouseSensitivity;
      freeCamera.pitch=Math.max(-1.25,Math.min(1.25,freeCamera.pitch-e.movementY*.002*mouseSensitivity));
    } else {
      player.yaw += e.movementX * .0024 * mouseSensitivity;
      player.pitch = Math.max(-1.15,Math.min(1.15,player.pitch - e.movementY*.002*mouseSensitivity));
    }
  });
  document.addEventListener("mousedown", e => {
    if(e.button===0 && running && !freeCamera.active && document.pointerLockElement===canvas) {
      mouseFiring=true;
      fire();
    }
  });
  document.addEventListener("mouseup",e=>{if(e.button===0) mouseFiring=false;});
  document.addEventListener("mousedown", e => { if (e.button===2 && running && !freeCamera.active && document.pointerLockElement===canvas) throwHeldItem(); });
  canvas.addEventListener("contextmenu",e=>e.preventDefault());
  document.addEventListener("keydown", e => {
    const k=e.key.toLowerCase();
    if (!cheatMenu.hidden || !levelSelect.hidden) return;
    if (["w","a","s","d","e","q","p","arrowup","arrowdown","arrowleft","arrowright"," "].includes(k)) e.preventDefault();
    if(!e.repeat && k==="f" && running && !finishPending) {
      e.preventDefault();
      toggleFreeCamera();
      return;
    }
    if (!e.repeat && k===" " && running && !freeCamera.active && player.y===0) {
      player.verticalVelocity=7.5;
      playSound("jump");
    }
    if (!e.repeat && k==="e" && !freeCamera.active) pickUp();
    if (!e.repeat && k==="q" && !freeCamera.active) possessAimedEnemy();
    if (!e.repeat && k==="p") {
      showEnemyHitboxes=!showEnemyHitboxes;
      toast.textContent=showEnemyHitboxes?"ENEMY HITBOXES ON":"ENEMY HITBOXES OFF";
      toast.classList.add("on");
      clearTimeout(toastTimer);
      toastTimer=setTimeout(()=>{toast.classList.remove("on");toast.textContent="SHOT FIRED";},700);
    }
    if (!e.repeat && k==="z" && finishPending) {
      finishPending=false;
      finishDeadline=0;
      showFinish(true);
    }
    held.add(k);
  });
  document.addEventListener("keyup", e => held.delete(e.key.toLowerCase()));
  window.addEventListener("blur",()=>held.clear());

  function frame(now) {
    resize();
    const dt=Math.max(0,Math.min((now-last)/1000,.05)); last=now;
    if(finishPending) {
      const remaining=Math.max(0,finishDeadline-now);
      document.querySelector("#timer").textContent=(remaining/1000).toFixed(1)+"s";
      if(remaining===0) {
        finishPending=false;
        showFinish(true);
      }
    }
    let replayFrame=null;
    if(finishPending && replayFrames.length) {
      const playbackTime=Math.max(0,now-replayStartedAt);
      let low=0, high=replayPlaybackTimes.length-1;
      while(low<high) {
        const middle=Math.ceil((low+high)/2);
        if(replayPlaybackTimes[middle]<=playbackTime) low=middle;
        else high=middle-1;
      }
      const frameIndex=low;
      replayFrame=replayFrames[frameIndex];
    }
    const forward=(held.has("w")||held.has("arrowup")?1:0)-(held.has("s")||held.has("arrowdown")?1:0);
    const strafe=(held.has("d")||held.has("arrowright")?1:0)-(held.has("a")||held.has("arrowleft")?1:0);
    const vertical=(held.has(" ")?1:0)-(held.has("control")||held.has("shift")?1:0);
    const moving=running && (freeCamera.active
      ? forward!==0 || strafe!==0 || vertical!==0
      : forward!==0 || strafe!==0 || player.y>0 || player.verticalVelocity>0);
    const timeScale=moving?1:.035;
    const replaySlowMotion=finishPending && replayFrame ? replayFrame.slowMotion : !moving;
    document.body.dataset.gameState=finishPending?"replay":running?"playing":"menu";
    document.body.dataset.motion=replaySlowMotion?"slow":"normal";
    motionCopy.textContent=replaySlowMotion?"TIME DILATION / ACTIVE":"KINETIC / NORMAL";
    motionDetail.textContent=replaySlowMotion?"WORLD FLOW REDUCED":"MOTION AT FULL SPEED";
    if(running && !freeCamera.active) {
      player.y=Math.max(0,player.y+player.verticalVelocity*dt);
      if(player.y===0) player.verticalVelocity=0;
      else player.verticalVelocity-=18*dt;
      elapsed+=dt;
      recordReplay(now);
      const seconds=Math.floor(elapsed);
      document.querySelector("#timer").textContent=
        String(Math.floor(seconds/60)).padStart(2,"0")+":"+String(seconds%60).padStart(2,"0");
    }
    if(running && !freeCamera.active && mouseFiring && weaponType==="ar" && now-lastShotAt>=(noReloadEnabled?90:2000)) fire();
    if (running && moving) {
      if(freeCamera.active) {
        const magnitude=Math.hypot(forward,strafe,vertical)||1;
        const speed=8*dt*(held.has("shift")?2.5:1);
        const fx=Math.sin(freeCamera.yaw)*Math.cos(freeCamera.pitch);
        const fy=Math.sin(freeCamera.pitch);
        const fz=-Math.cos(freeCamera.yaw)*Math.cos(freeCamera.pitch);
        const rx=Math.cos(freeCamera.yaw),rz=Math.sin(freeCamera.yaw);
        freeCamera.x+=(fx*forward+rx*strafe)/magnitude*speed;
        freeCamera.y+=(fy*forward+vertical)/magnitude*speed;
        freeCamera.z+=(fz*forward+rz*strafe)/magnitude*speed;
      } else {
        const mag=Math.hypot(forward,strafe)||1, speed=4.2*dt;
        const fx=Math.sin(player.yaw), fz=-Math.cos(player.yaw), rx=Math.cos(player.yaw), rz=Math.sin(player.yaw);
        const nx=player.x+(fx*forward+rx*strafe)/mag*speed;
        const nz=player.z+(fz*forward+rz*strafe)/mag*speed;
        if (!intersectsBlock(nx,player.z,.32,floorHeightAt(nx,player.z))) player.x=nx;
        if (!intersectsBlock(player.x,nz,.32,floorHeightAt(player.x,nz))) player.z=nz;
        for(const prop of props) {
          if(prop.kind!=="explosiveBall" || prop.thrown) continue;
          const dx=prop.x-player.x,dz=prop.z-player.z,distance=Math.hypot(dx,dz);
          if(distance<.72) {
            prop.vx=dx/(distance||1)*6;
            prop.vy=1.1;
            prop.vz=dz/(distance||1)*6;
            prop.thrown=true;
          }
        }
      }
      if(!freeCamera.active) {
        for (const e of enemies) if (e.alive) {
          let targetX=player.x,targetZ=player.z;
          const enemyFloor=floorHeightAt(e.x,e.z),playerFloor=floorHeightAt(player.x,player.z);
          if(levels[levelIndex].twoStory && Math.abs(enemyFloor-playerFloor)>1.25) {
            if(enemyFloor<playerFloor) {
              if(e.z>3.25) { targetX=-2.4;targetZ=2.7; }
              else { targetX=-2.4;targetZ=-3.8; }
            } else if(e.z<-3.65) { targetX=-2.4;targetZ=-3.4; }
            else { targetX=-2.4;targetZ=2.7; }
          }
          const dx=targetX-e.x, dz=targetZ-e.z, d=Math.hypot(dx,dz);
          if(d>.001) e.facing=Math.atan2(dx,dz);
          if (Math.hypot(player.x-e.x,player.z-e.z)<.78 && Math.abs(enemyFloor-playerFloor)<1.25 && player.y<.65) { finish(false); break; }
          if(d<.001) continue;
          const stepX=dx/d*dt*timeScale*1.35;
          const stepZ=dz/d*dt*timeScale*1.35;
          const oldX=e.x, oldZ=e.z;
          if(!intersectsBlock(e.x+stepX,e.z,.3,floorHeightAt(e.x+stepX,e.z))) e.x+=stepX;
          if(!intersectsBlock(e.x,e.z+stepZ,.3,floorHeightAt(e.x,e.z+stepZ))) e.z+=stepZ;
          const moveDistance=Math.hypot(e.x-oldX,e.z-oldZ);
          const expectedDistance=dt*timeScale*1.35;
          e.gaitAmount=Math.min(1,moveDistance/(expectedDistance||1));
          e.gaitPhase+=dt*timeScale*(2.4+e.gaitAmount*8.5);
          e.shootTimer-=dt*timeScale;
          if (e.shootTimer<=0) {
            fireEnemyShot(e,Math.atan2(player.x-e.x,player.z-e.z));
            e.shootTimer=1.45+Math.random()*.45;
          }
        }
        if(running) {
          for(const enemy of enemies) if(enemy.alive) {
            enemy.phase+=dt*timeScale*(moving?5:1.4);
            if(!moving) enemy.gaitAmount=Math.max(0,enemy.gaitAmount-dt*3.5);
          }
        }
      }
    }
    if (running) {
      for (let i=bullets.length-1;i>=0;i--) {
        const bullet=bullets[i], step=dt*timeScale;
        const distance=Math.hypot(bullet.vx,bullet.vy,bullet.vz)*step;
        const steps=Math.max(1,Math.ceil(distance/.08));
        let remove=false;
        for (let j=0;j<steps;j++) {
          const substep=step/steps;
          bullet.x+=bullet.vx*substep; bullet.y+=bullet.vy*substep; bullet.z+=bullet.vz*substep;
          if (intersectsBlock(bullet.x,bullet.z,.04,bullet.y) || bullet.y<.05) {
            if(bullet.weapon==="candyCane") explodeCandyCane(bullet);
            remove=true;
            break;
          }
          if (bullet.team==="hostile" && Math.hypot(bullet.x-player.x,bullet.z-player.z)<.38 && Math.abs(bullet.y-(floorHeightAt(player.x,player.z)+1.3+player.y))<.4) {
            finish(false); remove=true; break;
          }
          if (bullet.team==="player") {
            const hit=enemies.find(e=>e.alive && enemyHitboxContains(e,bullet.x,bullet.y,bullet.z));
            if (hit) {
              if(bullet.weapon==="candyCane") explodeCandyCane(bullet);
              else {
                defeatEnemy(hit);
                if (!enemies.some(e=>e.alive)) finish(true);
              }
              remove=true;
              break;
            }
          }
        }
        bullet.life-=step;
        if(bullet.weapon==="throwingStars") bullet.spin=(bullet.spin||0)+step*32;
        if (remove || bullet.life<=0) bullets.splice(i,1);
      }
      for(const prop of props) {
        if(!prop.thrown) continue;
        const step=dt*timeScale;
        const distance=Math.hypot(prop.vx,prop.vy,prop.vz)*step;
        const steps=Math.max(1,Math.ceil(distance/.06));
        for(let j=0;j<steps;j++) {
          const substep=step/steps;
          prop.vy-=5.5*substep;
          const x=prop.x+prop.vx*substep;
          const y=prop.y+prop.vy*substep;
          const z=prop.z+prop.vz*substep;
          const hit=enemies.find(enemy=>enemy.alive && Math.abs((prop.floorY||0)-floorHeightAt(enemy.x,enemy.z))<1.25 && enemyHitboxContains(enemy,x,y,z));
          if(hit) {
            if(prop.kind==="explosiveBall") {
              const blastTargets=enemies.filter(enemy=>enemy.alive && Math.abs((prop.floorY||0)-floorHeightAt(enemy.x,enemy.z))<1.25 && Math.hypot(x-enemy.x,z-enemy.z)<2.25);
              explode(x,y,z);
              playSound("hit");
              for(const enemy of blastTargets) defeatEnemy(enemy);
              prop.destroyed=true;
            } else {
              defeatEnemy(hit);
              prop.thrown=false;
              prop.vx=prop.vy=prop.vz=0;
            }
            if(!enemies.some(enemy=>enemy.alive)) finish(true);
            break;
          }
          if(intersectsBlock(x,z,.09,prop.floorY||0)) {
            prop.vx*=-.28;
            prop.vz*=-.28;
            prop.vy*=.45;
            break;
          }
          prop.x=x;
          prop.y=y;
          prop.z=z;
          prop.spin+=substep*14;
          if(prop.y<(prop.floorY||0)+.12) {
            prop.y=(prop.floorY||0)+.12;
            prop.vy=Math.abs(prop.vy)*.24;
            prop.vx*=.78;
            prop.vz*=.78;
            if(Math.abs(prop.vy)<.28 && Math.hypot(prop.vx,prop.vz)<.38) {
              prop.thrown=false;
              prop.vx=prop.vy=prop.vz=0;
            }
            break;
          }
        }
      }
      props=props.filter(prop=>!prop.destroyed);
      if (thrownGun) {
        const step=dt*timeScale;
        const distance=Math.hypot(thrownGun.vx,thrownGun.vy,thrownGun.vz)*step;
        const steps=Math.max(1,Math.ceil(distance/.08));
        for (let j=0;j<steps;j++) {
          const substep=step/steps;
          thrownGun.vy-=5.5*substep;
          const x=thrownGun.x+thrownGun.vx*substep;
          const y=thrownGun.y+thrownGun.vy*substep;
          const z=thrownGun.z+thrownGun.vz*substep;
          if (intersectsBlock(x,z,.1,thrownGun.floorY||0) || y<(thrownGun.floorY||0)+.12) {
            thrownGun.vx=0; thrownGun.vy=0; thrownGun.vz=0;
            thrownGun.y=Math.max((thrownGun.floorY||0)+.12,thrownGun.y);
            break;
          }
          thrownGun.x=x; thrownGun.y=y; thrownGun.z=z;
          thrownGun.spin+=substep*12;
          const hit=enemies.find(e=>e.alive && enemyHitboxContains(e,x,y,z));
          if (hit) {
            defeatEnemy(hit);
            thrownGun.vx=0; thrownGun.vy=0; thrownGun.vz=0;
            if (!enemies.some(e=>e.alive)) finish(true);
            break;
          }
        }
        thrownGun.life-=step;
        if (thrownGun.life<=0) { thrownGun.vx=0; thrownGun.vy=0; thrownGun.vz=0; }
      }
      for (let i=droppedGuns.length-1;i>=0;i--) {
        const gun=droppedGuns[i], step=dt*timeScale;
        gun.vy-=6*step;
        const x=gun.x+gun.vx*step, y=gun.y+gun.vy*step, z=gun.z+gun.vz*step;
        if(intersectsBlock(x,z,.12,gun.floorY||0)) { gun.vx=0; gun.vz=0; }
        else { gun.x=x; gun.z=z; }
        if(y<(gun.floorY||0)+.14) { gun.y=(gun.floorY||0)+.14; gun.vy=0; gun.vx*=.82; gun.vz*=.82; }
        else gun.y=y;
        gun.spin+=step*2;
        gun.life-=step;
        if(gun.life<=0) droppedGuns.splice(i,1);
      }
      for(let i=shards.length-1;i>=0;i--) {
        const shard=shards[i], step=dt*timeScale;
        shard.vy-=8*step;
        shard.x+=shard.vx*step; shard.y+=shard.vy*step; shard.z+=shard.vz*step;
        const shardFloor=shard.floorY||0;
        if(shard.y<shardFloor+.04) { shard.y=shardFloor+.04; shard.vy=Math.abs(shard.vy)*.22; shard.vx*=.72; shard.vz*=.72; }
        shard.spin+=step*8;
        shard.life-=step;
        if(shard.life<=0) shards.splice(i,1);
      }
      for(let i=explosions.length-1;i>=0;i--) {
        explosions[i].life-=dt*timeScale;
        if(explosions[i].life<=0) explosions.splice(i,1);
      }
    }
    const renderPlayer=replayFrame?.player||{
      ...player,weaponType,hasGun,swordSwingRemaining:Math.max(0,swordSwingUntil-now),punchSwingRemaining:Math.max(0,punchSwingUntil-now)
    };
    const renderEnemies=replayFrame?.enemies||enemies;
    const renderBullets=replayFrame?.bullets||bullets;
    const renderShards=replayFrame?.shards||shards;
    const renderExplosions=replayFrame?.explosions||explosions;
    const renderDroppedGuns=replayFrame?.droppedGuns||droppedGuns;
    const renderProps=replayFrame?.props||props;
    const renderHeldProp=replayFrame?replayFrame.heldProp:heldProp;
    const renderThrownGun=replayFrame?replayFrame.thrownGun:thrownGun;
    const renderFloorY=floorHeightAt(renderPlayer.x,renderPlayer.z);
    const cameraX=freeCamera.active?freeCamera.x:renderPlayer.x;
    const cameraY=freeCamera.active?freeCamera.y:renderFloorY+1.62+renderPlayer.y;
    const cameraZ=freeCamera.active?freeCamera.z:renderPlayer.z;
    const cameraYaw=freeCamera.active?freeCamera.yaw:renderPlayer.yaw;
    const cameraPitch=freeCamera.active?freeCamera.pitch:renderPlayer.pitch;
    const eye=[cameraX,cameraY,cameraZ];
    const direction=[Math.sin(cameraYaw)*Math.cos(cameraPitch),Math.sin(cameraPitch),-Math.cos(cameraYaw)*Math.cos(cameraPitch)];
    const target=[eye[0]+direction[0],eye[1]+direction[1],eye[2]+direction[2]];
    const view=lookAt(eye,target);
    const projection=perspective(Math.PI/3,canvas.width/canvas.height,.1,80);
    const clearShade=swappedPalette ? .96 : .008;
    gl.clearColor(clearShade,clearShade,clearShade,1); gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);
    const room=levels[levelIndex];
    drawRoomInterior(view,projection,room);
    for (const b of blocks) {
      drawBox(view,projection,b.x,.9,b.z,b.w/2,.9,b.d/2,[.9,.9,.9]);
      drawBox(view,projection,b.x,1.82,b.z,b.w/2,.035,b.d/2,[.82,.82,.82]);
    }
    for(const b of room.upperBlocks||[]) {
      drawBox(view,projection,b.x,3.9,b.z,b.w/2,.9,b.d/2,[.9,.9,.9]);
      drawBox(view,projection,b.x,4.82,b.z,b.w/2,.035,b.d/2,[.82,.82,.82]);
    }
    for(const prop of renderProps) {
      drawThrowableProp(view,projection,prop,prop.x,prop.y,prop.z,prop.spin||0);
    }
    if(wallHackEnabled) {
      gl.disable(gl.DEPTH_TEST);
      for(const enemy of renderEnemies) if(enemy.alive) {
        const angle=enemy.facing??Math.atan2(renderPlayer.x-enemy.x,renderPlayer.z-enemy.z);
        const bob=Math.sin(enemy.phase)*.022+Math.sin(enemy.phase*.47)*.009;
        const gaitPhase=enemy.gaitPhase||0,gaitAmount=enemy.gaitAmount||0;
        const stride=Math.sin(gaitPhase)*.22*gaitAmount;
        drawEnemyOutline(view,projection,enemy,angle,bob,stride,gaitPhase,gaitAmount,floorHeightAt(enemy.x,enemy.z));
      }
      gl.enable(gl.DEPTH_TEST);
    }
    for (const e of renderEnemies) if (e.alive) {
      const angle=e.facing??Math.atan2(renderPlayer.x-e.x,renderPlayer.z-e.z);
      const bob=Math.sin(e.phase)*.022+Math.sin(e.phase*.47)*.009;
      const gaitPhase=e.gaitPhase||0, gaitAmount=e.gaitAmount||0;
      const stride=Math.sin(gaitPhase)*.22*gaitAmount;
      const enemyFloor=floorHeightAt(e.x,e.z);
      const aiming=enemyCanSeePlayer(e,renderPlayer);
      drawEnemyModel(view,projection,e,angle,bob,stride,gaitPhase,gaitAmount,enemyFloor,aiming);
      const armSwing=stride*.72;
      const gunX=e.x+Math.cos(angle)*(.38+armSwing*.5)+Math.sin(angle)*(.25-armSwing*.25);
      const gunZ=e.z-Math.sin(angle)*(.38+armSwing*.5)+Math.cos(angle)*(.25-armSwing*.25);
      const gunY=enemyFloor+(aiming?1.02:.9)+bob+Math.abs(armSwing)*.12;
      const aimDx=renderPlayer.x-e.x,aimDz=renderPlayer.z-e.z;
      const aimDistance=Math.hypot(aimDx,aimDz);
      const aimAngle=aiming?Math.atan2(aimDx,aimDz):angle;
      const aimPitch=aiming
        ? Math.max(-.55,Math.min(.55,Math.atan2(renderFloorY+1.25+renderPlayer.y-gunY,aimDistance)+.16))
        : 0;
      drawWeapon(view,projection,e.weapon,gunX,gunY,gunZ,aimAngle+Math.PI,aimPitch,1);
      if(showEnemyHitboxes) drawEnemyHitbox(view,projection,e.x,enemyFloor+enemyHitboxHalfHeight+bob,e.z,enemyHitboxHalfWidth,enemyHitboxHalfHeight,enemyHitboxHalfDepth);
    }
    for (const bullet of renderBullets) drawTrail(view,projection,bullet);
    for(const shard of renderShards) {
      const size=shard.size*Math.max(.12,Math.min(1,shard.life*2));
      drawBox(view,projection,shard.x,shard.y,shard.z,size,size,size,shard.color,shard.spin);
    }
    for(const explosion of renderExplosions) {
      const progress=1-explosion.life/explosion.maxLife;
      const radius=.18+progress*1.05;
      const color=progress<.45?[1,.62,.12]:[1,.2,.035];
      drawSphere(view,projection,explosion.x,explosion.y,explosion.z,radius,radius*.82,radius,color);
    }
    for(const pickup of weaponPickups) {
      const bob=Math.sin(now*.003+pickup.phase)*.08;
      const color=pickup.weapon==="sword"?[.75,.8,.83]:pickup.weapon==="shotgun"?[.7,.42,.18]:[.2,.58,.72];
      const pickupFloor=floorHeightAt(pickup.x,pickup.z);
      drawSphere(view,projection,pickup.x,pickupFloor+.13,pickup.z,.28,.045,.28,color);
      drawWeapon(view,projection,pickup.weapon,pickup.x,pickupFloor+.38+bob,pickup.z,now*.0005);
    }
    for(const gun of renderDroppedGuns) {
      drawWeapon(view,projection,gun.weapon,gun.x,gun.y,gun.z,gun.spin);
    }
    if(renderHeldProp && !freeCamera.active) {
      const propX=renderPlayer.x+Math.cos(renderPlayer.yaw)*.38+Math.sin(renderPlayer.yaw)*.42;
      const propY=renderFloorY+1.2+renderPlayer.y+Math.sin(renderPlayer.pitch)*.2;
      const propZ=renderPlayer.z+Math.sin(renderPlayer.yaw)*.38-Math.cos(renderPlayer.yaw)*.42;
      drawThrowableProp(view,projection,renderHeldProp,propX,propY,propZ,-renderPlayer.yaw);
    }
    if(!renderPlayer.hasGun && !freeCamera.active) {
      const punchRemaining=renderPlayer.punchSwingRemaining||0;
      const punchProgress=Math.max(0,Math.min(1,punchRemaining/380));
      const extension=Math.sin((1-punchProgress)*Math.PI);
      const forwardOffset=.58+extension*.42;
      const rightOffset=.3;
      const forwardX=Math.sin(renderPlayer.yaw),forwardZ=-Math.cos(renderPlayer.yaw);
      const rightX=Math.cos(renderPlayer.yaw),rightZ=Math.sin(renderPlayer.yaw);
      const armX=renderPlayer.x+forwardX*forwardOffset+rightX*rightOffset;
      const armY=renderFloorY+1.13+renderPlayer.y+Math.sin(renderPlayer.pitch)*forwardOffset;
      const armZ=renderPlayer.z+forwardZ*forwardOffset+rightZ*rightOffset;
      const armRotation=Math.PI-renderPlayer.yaw;
      drawBox(view,projection,armX,armY,armZ,.13,.14,.38,[.42,.25,.18],armRotation,renderPlayer.pitch*.4);
      drawSphere(view,projection,armX+forwardX*.19,armY+Math.sin(renderPlayer.pitch)*.19,armZ+forwardZ*.19,.16,.145,.16,[.12,.1,.095],armRotation);
    }
    if (renderPlayer.hasGun && !freeCamera.active) {
      const weaponPitch=renderPlayer.weaponType==="sword"?0:renderPlayer.pitch;
      const swordEquipped=renderPlayer.weaponType==="sword";
      const candyCaneEquipped=renderPlayer.weaponType==="candyCane";
      const sideOffset=swordEquipped ? .38 : candyCaneEquipped ? .28 : .35;
      const forwardOffset=(swordEquipped ? .88 : candyCaneEquipped ? .9 : .45)*Math.cos(weaponPitch);
      const gunX=renderPlayer.x+Math.cos(renderPlayer.yaw)*sideOffset+Math.sin(renderPlayer.yaw)*forwardOffset;
      const gunY=renderFloorY+(swordEquipped ? 1.12 : candyCaneEquipped ? 1.55 : 1.22)+renderPlayer.y+.45*Math.sin(weaponPitch);
      const gunZ=renderPlayer.z+Math.sin(renderPlayer.yaw)*sideOffset-Math.cos(renderPlayer.yaw)*forwardOffset;
      const swingProgress=replayFrame
        ? Math.max(0,Math.min(1,1-renderPlayer.swordSwingRemaining/swordSwingDuration))
        : Math.max(0,Math.min(1,1-(swordSwingUntil-now)/swordSwingDuration));
      let swordYaw=0,swordPitch=0,swordRight=0,swordLift=0;
      if(swordEquipped && swingProgress>0 && swingProgress<1) {
        let frameIndex=0;
        while(frameIndex<swordSwingKeyframes.length-2 && swingProgress>swordSwingKeyframes[frameIndex+1][0]) frameIndex++;
        const from=swordSwingKeyframes[frameIndex],to=swordSwingKeyframes[frameIndex+1];
        const amount=smoothStep((swingProgress-from[0])/(to[0]-from[0]));
        swordYaw=from[1]+(to[1]-from[1])*amount;
        swordPitch=from[2]+(to[2]-from[2])*amount;
        swordRight=from[3]+(to[3]-from[3])*amount;
        swordLift=from[4]+(to[4]-from[4])*amount;
      }
      const weaponX=gunX+(swordEquipped?Math.cos(renderPlayer.yaw)*swordRight+Math.sin(renderPlayer.yaw)*.08:0);
      const weaponY=gunY+(swordEquipped?swordLift:0);
      const weaponZ=gunZ+(swordEquipped?Math.sin(renderPlayer.yaw)*swordRight-Math.cos(renderPlayer.yaw)*.08:0);
      const weaponRotation=swordEquipped?renderPlayer.yaw+swordYaw:-renderPlayer.yaw;
      drawWeapon(view,projection,renderPlayer.weaponType,weaponX,weaponY,weaponZ,weaponRotation,swordEquipped?swordPitch:weaponPitch);
    }
    if (renderThrownGun) {
      const angle=Math.atan2(renderThrownGun.vx,renderThrownGun.vz);
      drawWeapon(view,projection,renderThrownGun.weapon,renderThrownGun.x,renderThrownGun.y,renderThrownGun.z,renderThrownGun.spin||angle);
    }
    requestAnimationFrame(frame);
  }
  reset();
  requestAnimationFrame(frame);
})();
</script>
</body>
</html>

