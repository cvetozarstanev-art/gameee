<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Memory Game</title>
  <style>
    *{margin:0;padding:0;box-sizing:border-box}
    body{
      font-family:Arial, sans-serif;
      background: linear-gradient(135deg,#1a472a 0%, #0f3d1f 25%, #165a2c 50%, #0d3318 75%, #1e5335 100%);
      background-size:400% 400%;
      animation: gradientShift 15s ease infinite;
      min-height:100vh;
      display:flex;
      justify-content:center;
      align-items:center;
      padding:20px;
      overflow-x:hidden;
      color:#fff;
    }

    @keyframes gradientShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}

    .entry-menu{
      position:fixed;inset:0;
      background: linear-gradient(135deg, #c41e3a 0%, #165a4c 50%, #1e5347 100%);
      background-size:400% 400%;
      animation: gradientShift 10s ease infinite;
      display:flex;align-items:center;justify-content:center;
      z-index:2000;
      transition: opacity .5s, transform .5s;
    }
    .entry-menu.hidden{opacity:0;transform:scale(.95);pointer-events:none}

    .menu-content{text-align:center;position:relative;animation: menuFadeIn 1s ease-out}
    @keyframes menuFadeIn{from{opacity:0;transform:translateY(-30px)}to{opacity:1;transform:translateY(0)}}

    .snow-container{position:fixed;inset:0;pointer-events:none;overflow:hidden}
    .snowflake{position:absolute;top:-50px;color:white;font-size:18px;opacity:.8;animation: fall linear infinite}
    .snowflake:nth-child(1){left:10%;animation-duration:8s;animation-delay:0s}
    .snowflake:nth-child(2){left:20%;animation-duration:10s;animation-delay:1s}
    .snowflake:nth-child(3){left:30%;animation-duration:7s;animation-delay:2s}
    .snowflake:nth-child(4){left:40%;animation-duration:9s;animation-delay:.5s}
    .snowflake:nth-child(5){left:50%;animation-duration:11s;animation-delay:1.5s}
    .snowflake:nth-child(6){left:60%;animation-duration:8.5s;animation-delay:.8s}
    .snowflake:nth-child(7){left:70%;animation-duration:9.5s;animation-delay:1.2s}
    .snowflake:nth-child(8){left:80%;animation-duration:10.5s;animation-delay:.3s}
    .snowflake:nth-child(9){left:90%;animation-duration:7.5s;animation-delay:1.8s}
    .snowflake:nth-child(10){left:95%;animation-duration:9.8s;animation-delay:.6s}
    @keyframes fall{to{transform:translateY(100vh) rotate(360deg)}}

    .menu-title{
      font-size: 3rem;
      text-shadow:0 0 10px rgba(255,255,255,.8),0 0 20px rgba(255,215,0,.6), 3px 3px 6px rgba(0,0,0,.5);
      margin-bottom:10px;
      animation:titlePulse 2s ease-in-out infinite;
    }
    @keyframes titlePulse{0%,100%{transform:scale(1)}50%{transform:scale(1.05)}}
    .menu-subtitle{opacity:.95;margin-bottom:30px}
    .menu-buttons{display:flex;flex-direction:column;gap:14px;align-items:center;margin-bottom:18px}
    .menu-btn{
      background: linear-gradient(135deg, #fff 0%, #f0f0f0 100%);
      border:4px solid #d4af37;
      padding:16px 28px;
      border-radius:18px;
      cursor:pointer;
      color:#1b2a2a;
      transition: all .25s;
      box-shadow:0 8px 16px rgba(0,0,0,.35);
      min-width:320px;
      display:flex;align-items:center;justify-content:center;gap:12px;
    }
    .menu-btn:hover{transform:translateY(-3px) scale(1.02);border-color:#ffd700}
    .btn-icon{font-weight:bold;font-size:18px}
    .btn-text{font-weight:bold;font-size:18px}
    .btn-detail{font-size:12px;opacity:.85}

    .music-btn{
      background: rgba(255,255,255,.18);
      border:2px solid rgba(255,255,255,.5);
      color:#fff;
      padding:10px 22px;
      border-radius:28px;
      cursor:pointer;
      transition: all .25s;
      backdrop-filter: blur(10px);
    }
    .music-btn.playing{background: rgba(76,175,80,.45);border-color:#4CAF50}

    .container{max-width:900px;width:100%;text-align:center}
    .container.hidden{display:none}

    .header-controls{display:flex;gap:10px;align-items:center;justify-content:space-between;margin-bottom:18px}
    .back-btn{
      background: rgba(255,255,255,.18);
      border:2px solid rgba(255,255,255,.5);
      color:#fff;
      padding:10px 16px;
      border-radius:10px;
      cursor:pointer;
      backdrop-filter: blur(10px);
      transition: all .2s;
    }
    .back-btn:hover{transform:translateX(-3px)}

    h1{font-size:2rem;flex:1;text-shadow:0 0 10px rgba(255,255,255,.6), 2px 2px 4px rgba(0,0,0,.3)}

    .scoreboard{
      background: rgba(255,255,255,.95);
      border-radius:15px;
      padding:16px;
      margin-bottom:18px;
      display:flex;
      justify-content:space-around;
      align-items:center;
      color:#165a4c;
      box-shadow:0 8px 16px rgba(0,0,0,.3);
      flex-wrap:wrap;
      gap:10px;
    }
    .score-item{font-weight:bold}
    .score-item .value{color:#d32f2f;font-size:22px;margin-left:6px}

    .game-board{
      display:grid;
      gap:12px;
      margin-bottom:18px;
      padding:16px;
      border-radius:15px;
      background: rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(10px);
    }
    .game-board.easy{grid-template-columns:repeat(4,1fr)}
    .game-board.medium{grid-template-columns:repeat(5,1fr)}
    .game-board.hard{grid-template-columns:repeat(6,1fr)}

    .card{
      aspect-ratio:1;
      border-radius:10px;
      cursor:pointer;
      position:relative;
      transform-style:preserve-3d;
      transition:transform .6s;
      box-shadow:0 4px 8px rgba(0,0,0,.2);
    }
    .card:hover:not(.flipped):not(.matched){transform:scale(1.03)}
    .card.flipped{transform:rotateY(180deg)}

    .card-face,.card-back{
      position:absolute;inset:0;
      backface-visibility:hidden;
      display:flex;align-items:center;justify-content:center;
      border-radius:10px;
      font-size:42px;
      font-weight:bold;
      user-select:none;
    }
    .card-back{
      background: linear-gradient(135deg, #d32f2f 0%, #b71c1c 100%);
      box-shadow: inset 0 0 20px rgba(0,0,0,.2);
    }
    .card-face{
      background: linear-gradient(135deg, #fff 0%, #f5f5f5 100%);
      color:#1b2a2a;
      transform:rotateY(180deg);
    }

    .card.matched{cursor:default;opacity:1}
    .card.matched .card-face{
      background: linear-gradient(135deg, #7CFC00 0%, #32CD32 100%);
      color:#0d3318;
      box-shadow: inset 0 0 20px rgba(0,0,0,.15);
    }

    .restart-btn{
      background: linear-gradient(135deg, #d32f2f 0%, #b71c1c 100%);
      color:#fff;
      border:none;
      padding:12px 28px;
      border-radius:28px;
      cursor:pointer;
      font-weight:bold;
      box-shadow:0 4px 8px rgba(0,0,0,.3);
    }

    .modal{position:fixed;inset:0;background:rgba(0,0,0,.8);display:flex;align-items:center;justify-content:center;z-index:1000}
    .modal.hidden{display:none}
    .modal-content{background:#fff;color:#0d3d33;border-radius:18px;padding:28px;max-width:420px;width:92%;box-shadow:0 10px 30px rgba(0,0,0,.5)}
    .highlight{color:#d32f2f;font-weight:bold}
    .modal-buttons{display:flex;gap:10px;justify-content:center;flex-wrap:wrap;margin-top:12px}
    .modal-btn{background:linear-gradient(135deg,#165a4c 0%, #0d3d33 100%);color:#fff;border:none;padding:12px 20px;border-radius:22px;cursor:pointer;font-weight:bold}
    .modal-btn.secondary{background:linear-gradient(135deg,#757575 0%, #616161 100%)}

    @media (max-width:768px){
      .menu-title{font-size:2.2rem}
      .menu-btn{min-width:260px}
      h1{font-size:1.5rem}
      .card-face,.card-back{font-size:30px}
    }
  </style>
</head>
<body>
  <!-- Entry Menu -->
  <div id="entry-menu" class="entry-menu">
    <div class="snow-container">
      <div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div>
      <div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div><div class="snowflake">*</div>
    </div>
    <div class="menu-content">
      <h1 class="menu-title">Memory Game</h1>
      <p class="menu-subtitle">Find all matching pairs.</p>

      <div class="menu-buttons">
        <button id="start-easy" class="menu-btn"><span class="btn-icon">1</span><span class="btn-text">Easy</span><span class="btn-detail">4x4 (8 pairs)</span></button>
        <button id="start-medium" class="menu-btn"><span class="btn-icon">2</span><span class="btn-text">Medium</span><span class="btn-detail">4x5 (10 pairs)</span></button>
        <button id="start-hard" class="menu-btn"><span class="btn-icon">3</span><span class="btn-text">Hard</span><span class="btn-detail">6x6 (18 pairs)</span></button>
      </div>

      <button id="music-toggle" class="music-btn">Music Off</button>
    </div>
  </div>

  <!-- Game -->
  <div id="game-container" class="container hidden">
    <div class="header-controls">
      <button id="back-to-menu" class="back-btn">Back</button>
      <h1>Memory Game</h1>
      <button id="music-toggle-game" class="music-btn">Music Off</button>
    </div>

    <div class="scoreboard">
      <div class="score-item">Moves:<span id="moves" class="value">0</span></div>
      <div class="score-item">Time:<span id="timer" class="value">0</span> sec</div>
      <div class="score-item">Level:<span id="difficulty-display" class="value">Easy</span></div>
    </div>

    <div id="game-board" class="game-board easy"></div>

    <button id="restart-btn" class="restart-btn">New game</button>

    <div id="win-modal" class="modal hidden">
      <div class="modal-content">
        <h2>Nice!</h2>
        <p>You finished the game.</p>
        <p>Level: <span id="final-difficulty" class="highlight">Easy</span></p>
        <p>Moves: <span id="final-moves" class="highlight">0</span></p>
        <p>Time: <span id="final-time" class="highlight">0</span> sec</p>
        <div class="modal-buttons">
          <button id="modal-restart-btn" class="modal-btn">Play again</button>
          <button id="modal-menu-btn" class="modal-btn secondary">Menu</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // --- Music (WebAudio) ---
    class MusicPlayer {
      constructor(){this.audioContext=null;this.isPlaying=false;this.oscillators=[];this.gainNode=null;}
      init(){
        if(!this.audioContext){
          this.audioContext = new (window.AudioContext||window.webkitAudioContext)();
          this.gainNode = this.audioContext.createGain();
          this.gainNode.connect(this.audioContext.destination);
          this.gainNode.gain.value = 0.1;
        }
      }
      getMelody(){
        return [
          {note:'E4',duration:0.25},{note:'E4',duration:0.25},{note:'E4',duration:0.5},
          {note:'E4',duration:0.25},{note:'E4',duration:0.25},{note:'E4',duration:0.5},
          {note:'E4',duration:0.25},{note:'G4',duration:0.25},{note:'C4',duration:0.25},{note:'D4',duration:0.25},{note:'E4',duration:1},
          {note:'rest',duration:0.25},
          {note:'F4',duration:0.25},{note:'F4',duration:0.25},{note:'F4',duration:0.25},{note:'F4',duration:0.25},{note:'F4',duration:0.25},
          {note:'E4',duration:0.25},{note:'E4',duration:0.25},{note:'E4',duration:0.125},{note:'E4',duration:0.125},{note:'E4',duration:0.25},
          {note:'D4',duration:0.25},{note:'D4',duration:0.25},{note:'E4',duration:0.25},{note:'D4',duration:0.5},{note:'G4',duration:0.5}
        ];
      }
      getFrequency(note){
        const notes={C4:261.63,D4:293.66,E4:329.63,F4:349.23,G4:392.0,A4:440.0,B4:493.88,C5:523.25};
        return notes[note]||0;
      }
      sleep(ms){return new Promise(r=>setTimeout(r,ms));}
      playNote(note,duration){
        return new Promise((resolve)=>{
          const freq=this.getFrequency(note);
          if(!freq){resolve();return;}
          const osc=this.audioContext.createOscillator();
          const g=this.audioContext.createGain();
          osc.connect(g); g.connect(this.gainNode);
          osc.frequency.value=freq;
          osc.type='sine';
          const now=this.audioContext.currentTime;
          g.gain.setValueAtTime(0,now);
          g.gain.linearRampToValueAtTime(0.3,now+0.01);
          g.gain.exponentialRampToValueAtTime(0.01,now+duration);
          osc.start(now);
          osc.stop(now+duration);
          this.oscillators.push(osc);
          setTimeout(()=>{
            const idx=this.oscillators.indexOf(osc);
            if(idx>-1) this.oscillators.splice(idx,1);
            resolve();
          }, duration*1000);
        });
      }
      async loop(){
        if(!this.audioContext) return;
        const melody=this.getMelody();
        const tempo=120;
        const beat=60/tempo;
        while(this.isPlaying){
          for(const n of melody){
            if(!this.isPlaying) return;
            if(n.note==='rest') await this.sleep(n.duration*beat*1000);
            else await this.playNote(n.note, n.duration*beat);
          }
        }
      }
      start(){
        this.init();
        if(this.audioContext.state==='suspended') this.audioContext.resume();
        if(this.isPlaying) return;
        this.isPlaying=true;
        this.loop();
      }
      stop(){
        this.isPlaying=false;
        for(const osc of this.oscillators){ try{osc.stop();}catch{} }
        this.oscillators=[];
      }
    }

    // --- Game ---
    let cards=[];
    let flippedCards=[];
    let matchedPairs=0;
    let moves=0;
    let timer=0;
    let timerInterval=null;
    let isChecking=false;
    let gameStarted=false;
    let currentDifficulty='easy';

    const allSymbols = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('');
    const difficulties = {
      easy: { pairs: 8, gridClass: 'easy', name: 'Easy' },
      medium: { pairs: 10, gridClass: 'medium', name: 'Medium' },
      hard: { pairs: 18, gridClass: 'hard', name: 'Hard' }
    };

    const entryMenu=document.getElementById('entry-menu');
    const gameContainer=document.getElementById('game-container');
    const gameBoard=document.getElementById('game-board');
    const movesDisplay=document.getElementById('moves');
    const timerDisplay=document.getElementById('timer');
    const difficultyDisplay=document.getElementById('difficulty-display');
    const restartBtn=document.getElementById('restart-btn');
    const backToMenuBtn=document.getElementById('back-to-menu');
    const winModal=document.getElementById('win-modal');
    const finalDifficultyDisplay=document.getElementById('final-difficulty');
    const finalMovesDisplay=document.getElementById('final-moves');
    const finalTimeDisplay=document.getElementById('final-time');
    const modalRestartBtn=document.getElementById('modal-restart-btn');
    const modalMenuBtn=document.getElementById('modal-menu-btn');

    const startEasyBtn=document.getElementById('start-easy');
    const startMediumBtn=document.getElementById('start-medium');
    const startHardBtn=document.getElementById('start-hard');

    const musicToggleMenu=document.getElementById('music-toggle');
    const musicToggleGame=document.getElementById('music-toggle-game');
    const musicPlayer = new MusicPlayer();
    let isMusicPlaying=false;

    function toggleMusic(){
      if(isMusicPlaying){
        musicPlayer.stop();
        isMusicPlaying=false;
        musicToggleMenu.textContent='Music Off';
        musicToggleGame.textContent='Music Off';
        musicToggleMenu.classList.remove('playing');
        musicToggleGame.classList.remove('playing');
      }else{
        musicPlayer.start();
        isMusicPlaying=true;
        musicToggleMenu.textContent='Music On';
        musicToggleGame.textContent='Music On';
        musicToggleMenu.classList.add('playing');
        musicToggleGame.classList.add('playing');
      }
    }

    function startGame(difficulty){
      currentDifficulty=difficulty;
      entryMenu.classList.add('hidden');
      gameContainer.classList.remove('hidden');
      difficultyDisplay.textContent=difficulties[difficulty].name;
      initGame();
    }

    function backToMenu(){
      gameContainer.classList.add('hidden');
      entryMenu.classList.remove('hidden');
      stopGame();
    }

    function stopGame(){
      if(timerInterval){clearInterval(timerInterval);timerInterval=null;}
      gameStarted=false;
    }

    function initGame(){
      cards=[];flippedCards=[];matchedPairs=0;moves=0;timer=0;isChecking=false;gameStarted=false;
      if(timerInterval){clearInterval(timerInterval);timerInterval=null;}
      movesDisplay.textContent='0';
      timerDisplay.textContent='0';
      winModal.classList.add('hidden');

      const numPairs=difficulties[currentDifficulty].pairs;
      const selected=allSymbols.slice(0,numPairs);
      const symbols=[...selected,...selected];
      shuffle(symbols);
      cards=symbols.map((s,i)=>({id:i,symbol:s,flipped:false,matched:false}));

      gameBoard.className=`game-board ${difficulties[currentDifficulty].gridClass}`;
      renderBoard();
    }

    function shuffle(arr){
      for(let i=arr.length-1;i>0;i--){
        const j=Math.floor(Math.random()*(i+1));
        [arr[i],arr[j]]=[arr[j],arr[i]];
      }
    }

    function renderBoard(){
      gameBoard.innerHTML='';
      cards.forEach(card=>{
        const el=document.createElement('div');
        el.className='card';
        el.dataset.id=card.id;

        const back=document.createElement('div');
        back.className='card-back';
        back.textContent='';

        const face=document.createElement('div');
        face.className='card-face';
        face.textContent=card.symbol;

        el.appendChild(back);
        el.appendChild(face);
        el.addEventListener('click',()=>handleCardClick(card.id));
        gameBoard.appendChild(el);
      });
    }

    function startTimer(){
      if(!gameStarted){
        gameStarted=true;
        timerInterval=setInterval(()=>{timer++;timerDisplay.textContent=String(timer);},1000);
      }
    }

    function handleCardClick(cardId){
      if(isChecking) return;
      const card=cards[cardId];
      if(card.flipped||card.matched) return;

      startTimer();
      card.flipped=true;
      flippedCards.push(card);
      document.querySelector(`[data-id="${cardId}"]`).classList.add('flipped');

      if(flippedCards.length===2){
        isChecking=true;
        moves++;movesDisplay.textContent=String(moves);
        checkMatch();
      }
    }

    function checkMatch(){
      const [c1,c2]=flippedCards;
      if(c1.symbol===c2.symbol && c1.id!==c2.id){
        c1.matched=true;c2.matched=true;matchedPairs++;
        const e1=document.querySelector(`[data-id="${c1.id}"]`);
        const e2=document.querySelector(`[data-id="${c2.id}"]`);
        e1.classList.add('matched','flipped');
        e2.classList.add('matched','flipped');
        flippedCards=[];isChecking=false;
        if(matchedPairs===difficulties[currentDifficulty].pairs) endGame();
      }else{
        setTimeout(()=>{
          c1.flipped=false;c2.flipped=false;
          document.querySelector(`[data-id="${c1.id}"]`).classList.remove('flipped');
          document.querySelector(`[data-id="${c2.id}"]`).classList.remove('flipped');
          flippedCards=[];isChecking=false;
        },1000);
      }
    }

    function endGame(){
      if(timerInterval){clearInterval(timerInterval);timerInterval=null;}
      finalDifficultyDisplay.textContent=difficulties[currentDifficulty].name;
      finalMovesDisplay.textContent=String(moves);
      finalTimeDisplay.textContent=String(timer);
      setTimeout(()=>winModal.classList.remove('hidden'),300);
    }

    startEasyBtn.addEventListener('click',()=>startGame('easy'));
    startMediumBtn.addEventListener('click',()=>startGame('medium'));
    startHardBtn.addEventListener('click',()=>startGame('hard'));

    restartBtn.addEventListener('click', initGame);
    backToMenuBtn.addEventListener('click', backToMenu);
    modalRestartBtn.addEventListener('click', ()=>{winModal.classList.add('hidden');initGame();});
    modalMenuBtn.addEventListener('click', ()=>{winModal.classList.add('hidden');backToMenu();});

    musicToggleMenu.addEventListener('click', toggleMusic);
    musicToggleGame.addEventListener('click', toggleMusic);

    // Default music state shown
    musicToggleMenu.textContent='Music Off';
    musicToggleGame.textContent='Music Off';
  </script>
</body>
</html>
