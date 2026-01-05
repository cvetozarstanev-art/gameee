<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <None Update="wwwroot\**\*">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>

</Project>



// Simple Christmas melody generator using Web Audio API
class ChristmasMusicPlayer {
    constructor() {
        this.audioContext = null;
        this.isPlaying = false;
        this.oscillators = [];
        this.gainNode = null;
    }

    init() {
        if (!this.audioContext) {
            this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
            this.gainNode = this.audioContext.createGain();
            this.gainNode.connect(this.audioContext.destination);
            this.gainNode.gain.value = 0.1;
        }
    }

    // Simple "Jingle Bells" melody notes
    getMelody() {
        return [
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.5 },
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.5 },
            { note: 'E4', duration: 0.25 },
            { note: 'G4', duration: 0.25 },
            { note: 'C4', duration: 0.25 },
            { note: 'D4', duration: 0.25 },
            { note: 'E4', duration: 1 },
            { note: 'rest', duration: 0.25 },
            { note: 'F4', duration: 0.25 },
            { note: 'F4', duration: 0.25 },
            { note: 'F4', duration: 0.25 },
            { note: 'F4', duration: 0.25 },
            { note: 'F4', duration: 0.25 },
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.25 },
            { note: 'E4', duration: 0.125 },
            { note: 'E4', duration: 0.125 },
            { note: 'E4', duration: 0.25 },
            { note: 'D4', duration: 0.25 },
            { note: 'D4', duration: 0.25 },
            { note: 'E4', duration: 0.25 },
            { note: 'D4', duration: 0.5 },
            { note: 'G4', duration: 0.5 }
        ];
    }

    // Note frequencies (Hz)
    getFrequency(note) {
        const notes = {
            'C4': 261.63,
            'D4': 293.66,
            'E4': 329.63,
            'F4': 349.23,
            'G4': 392.00,
            'A4': 440.00,
            'B4': 493.88,
            'C5': 523.25
        };
        return notes[note] || 0;
    }

    async playMelodyLoop() {
        if (!this.audioContext) return;

        const melody = this.getMelody();
        const tempo = 120; // BPM
        const beatDuration = 60 / tempo;

        while (this.isPlaying) {
            for (const note of melody) {
                if (!this.isPlaying) return;

                if (note.note !== 'rest') {
                    await this.playNote(note.note, note.duration * beatDuration);
                } else {
                    await this.sleep(note.duration * beatDuration * 1000);
                }
            }
        }
    }

    playNote(note, duration) {
        return new Promise((resolve) => {
            const frequency = this.getFrequency(note);
            if (frequency === 0) {
                resolve();
                return;
            }

            const oscillator = this.audioContext.createOscillator();
            const noteGain = this.audioContext.createGain();

            oscillator.connect(noteGain);
            noteGain.connect(this.gainNode);

            oscillator.frequency.value = frequency;
            oscillator.type = 'sine';

            // Envelope
            const now = this.audioContext.currentTime;
            noteGain.gain.setValueAtTime(0, now);
            noteGain.gain.linearRampToValueAtTime(0.3, now + 0.01);
            noteGain.gain.exponentialRampToValueAtTime(0.01, now + duration);

            oscillator.start(now);
            oscillator.stop(now + duration);

            this.oscillators.push(oscillator);

            setTimeout(() => {
                const index = this.oscillators.indexOf(oscillator);
                if (index > -1) {
                    this.oscillators.splice(index, 1);
                }
                resolve();
            }, duration * 1000);
        });
    }

    sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    start() {
        this.init();
        if (this.audioContext.state === 'suspended') {
            this.audioContext.resume();
        }

        if (this.isPlaying) return;
        this.isPlaying = true;
        this.playMelodyLoop();
    }

    stop() {
        this.isPlaying = false;
        this.oscillators.forEach(osc => {
            try {
                osc.stop();
            } catch {
                // ignore
            }
        });
        this.oscillators = [];
    }
}

// Export for use in main script
if (typeof module !== 'undefined' && module.exports) {
    module.exports = ChristmasMusicPlayer;
}





// Game state
let cards = [];
let flippedCards = [];
let matchedPairs = 0;
let moves = 0;
let timer = 0;
let timerInterval = null;
let isChecking = false;
let gameStarted = false;
let currentDifficulty = 'easy';

// Symbols - using very widely supported characters
const allSymbols = [
    '🎅', '🎅🏼', '🎅🏽', '🎅🏾', '🎅🏿', '🤶', '🤶🏼', '🤶🏽',
    '🤶🏾', '🤶🏿', '❄️', '🎄', '⛄', '🕯️', '🔔', '🎁',
    '🧦', '🔥', '🌨️', '🎇'
];

// Difficulty settings (ASCII names)
const difficulties = {
    easy: { pairs: 8, gridClass: 'easy', name: 'Easy' },
    medium: { pairs: 10, gridClass: 'medium', name: 'Medium' },
    hard: { pairs: 18, gridClass: 'hard', name: 'Hard' }
};

// DOM elements
const entryMenu = document.getElementById('entry-menu');
const gameContainer = document.getElementById('game-container');
const gameBoard = document.getElementById('game-board');
const movesDisplay = document.getElementById('moves');
const timerDisplay = document.getElementById('timer');
const difficultyDisplay = document.getElementById('difficulty-display');
const restartBtn = document.getElementById('restart-btn');
const backToMenuBtn = document.getElementById('back-to-menu');
const winModal = document.getElementById('win-modal');
const finalDifficultyDisplay = document.getElementById('final-difficulty');
const finalMovesDisplay = document.getElementById('final-moves');
const finalTimeDisplay = document.getElementById('final-time');
const modalRestartBtn = document.getElementById('modal-restart-btn');
const modalMenuBtn = document.getElementById('modal-menu-btn');

// Menu buttons
const startEasyBtn = document.getElementById('start-easy');
const startMediumBtn = document.getElementById('start-medium');
const startHardBtn = document.getElementById('start-hard');

// Music elements
const musicToggleMenu = document.getElementById('music-toggle');
const musicToggleGame = document.getElementById('music-toggle-game');
let isMusicPlaying = false;
let musicPlayer = null;

// Initialize music player
function initMusic() {
    if (typeof ChristmasMusicPlayer !== 'undefined') {
        musicPlayer = new ChristmasMusicPlayer();
    }
}

// Toggle music
function toggleMusic() {
    if (!musicPlayer) return;

    if (isMusicPlaying) {
        musicPlayer.stop();
        isMusicPlaying = false;
        musicToggleMenu.textContent = 'Music Off';
        musicToggleGame.textContent = 'Music Off';
        musicToggleMenu.classList.remove('playing');
        musicToggleGame.classList.remove('playing');
    } else {
        musicPlayer.start();
        isMusicPlaying = true;
        musicToggleMenu.textContent = 'Music On';
        musicToggleGame.textContent = 'Music On';
        musicToggleMenu.classList.add('playing');
        musicToggleGame.classList.add('playing');
    }
}

// Start game with difficulty
function startGame(difficulty) {
    currentDifficulty = difficulty;
    entryMenu.classList.add('hidden');
    gameContainer.classList.remove('hidden');
    difficultyDisplay.textContent = difficulties[difficulty].name;
    initGame();
}

// Back to menu
function backToMenu() {
    gameContainer.classList.add('hidden');
    entryMenu.classList.remove('hidden');
    stopGame();
}

// Stop game
function stopGame() {
    if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
    }
    gameStarted = false;
}

// Initialize game
function initGame() {
    // Reset game state
    cards = [];
    flippedCards = [];
    matchedPairs = 0;
    moves = 0;
    timer = 0;
    isChecking = false;
    gameStarted = false;

    // Stop timer
    if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
    }

    // Update displays
    movesDisplay.textContent = '0';
    timerDisplay.textContent = '0';

    // Hide modal
    winModal.classList.add('hidden');

    // Get symbols for current difficulty
    const numPairs = difficulties[currentDifficulty].pairs;
    const selectedSymbols = allSymbols.slice(0, numPairs);

    // Create pairs - each symbol appears exactly twice
    const cardSymbols = [...selectedSymbols, ...selectedSymbols];

    // Shuffle cards (Fisher-Yates shuffle)
    shuffleArray(cardSymbols);

    // Create card objects with unique IDs
    cards = cardSymbols.map((symbol, index) => ({
        id: index,
        symbol: symbol,
        flipped: false,
        matched: false
    }));

    // Set grid class
    gameBoard.className = `game-board ${difficulties[currentDifficulty].gridClass}`;

    // Render game board
    renderBoard();
}

// Fisher-Yates shuffle algorithm
function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

// Render game board
function renderBoard() {
    gameBoard.innerHTML = '';

    cards.forEach(card => {
        const cardElement = document.createElement('div');
        cardElement.classList.add('card');
        cardElement.dataset.id = card.id;

        const cardBack = document.createElement('div');
        cardBack.classList.add('card-back');
        cardBack.textContent = '';

        const cardFace = document.createElement('div');
        cardFace.classList.add('card-face');
        cardFace.textContent = card.symbol;

        cardElement.appendChild(cardBack);
        cardElement.appendChild(cardFace);

        cardElement.addEventListener('click', () => handleCardClick(card.id));

        gameBoard.appendChild(cardElement);
    });
}

// Start timer on first click
function startTimer() {
    if (!gameStarted) {
        gameStarted = true;
        timerInterval = setInterval(() => {
            timer++;
            timerDisplay.textContent = timer;
        }, 1000);
    }
}

// Handle card click
function handleCardClick(cardId) {
    // Prevent clicks during checking or if card is already flipped/matched
    if (isChecking) return;

    const card = cards[cardId];
    if (card.flipped || card.matched) return;

    // Start timer on first click
    startTimer();

    // Flip card
    card.flipped = true;
    flippedCards.push(card);

    const cardElement = document.querySelector(`[data-id="${cardId}"]`);
    cardElement.classList.add('flipped');

    // Check if two cards are flipped
    if (flippedCards.length === 2) {
        isChecking = true;
        moves++;
        movesDisplay.textContent = moves;

        checkMatch();
    }
}

// Check if two flipped cards match
function checkMatch() {
    const [card1, card2] = flippedCards;

    if (card1.symbol === card2.symbol && card1.id !== card2.id) {
        // Match found
        card1.matched = true;
        card2.matched = true;
        matchedPairs++;

        const card1Element = document.querySelector(`[data-id="${card1.id}"]`);
        const card2Element = document.querySelector(`[data-id="${card2.id}"]`);

        card1Element.classList.add('matched');
        card2Element.classList.add('matched');

        // Ensure they remain visible (stay flipped)
        // They are already flipped; keep that state and prevent further interaction.
        card1Element.classList.add('flipped');
        card2Element.classList.add('flipped');

        flippedCards = [];
        isChecking = false;

        // Check if game is won
        if (matchedPairs === difficulties[currentDifficulty].pairs) {
            endGame();
        }
    } else {
        // No match - flip cards back after delay
        setTimeout(() => {
            card1.flipped = false;
            card2.flipped = false;

            const card1Element = document.querySelector(`[data-id="${card1.id}"]`);
            const card2Element = document.querySelector(`[data-id="${card2.id}"]`);

            card1Element.classList.remove('flipped');
            card2Element.classList.remove('flipped');

            flippedCards = [];
            isChecking = false;
        }, 1000);
    }
}

// End game
function endGame() {
    // Stop timer
    if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
    }

    // Show win modal
    finalDifficultyDisplay.textContent = difficulties[currentDifficulty].name;
    finalMovesDisplay.textContent = moves;
    finalTimeDisplay.textContent = timer;

    setTimeout(() => {
        winModal.classList.remove('hidden');
    }, 500);
}

// Event listeners
startEasyBtn.addEventListener('click', () => startGame('easy'));
startMediumBtn.addEventListener('click', () => startGame('medium'));
startHardBtn.addEventListener('click', () => startGame('hard'));

restartBtn.addEventListener('click', initGame);
backToMenuBtn.addEventListener('click', backToMenu);
modalRestartBtn.addEventListener('click', () => {
    winModal.classList.add('hidden');
    initGame();
});
modalMenuBtn.addEventListener('click', () => {
    winModal.classList.add('hidden');
    backToMenu();
});

musicToggleMenu.addEventListener('click', toggleMusic);
musicToggleGame.addEventListener('click', toggleMusic);

// Initialize music on page load
initMusic();






* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #1a472a 0%, #0f3d1f 25%, #165a2c 50%, #0d3318 75%, #1e5335 100%);
    background-size: 400% 400%;
    animation: gradientShift 15s ease infinite;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
    overflow-x: hidden;
}

@keyframes gradientShift {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

/* Entry Menu Styles */
.entry-menu {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(135deg, #c41e3a 0%, #165a4c 50%, #1e5347 100%);
    background-size: 400% 400%;
    animation: gradientShift 10s ease infinite;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2000;
    transition: opacity 0.5s, transform 0.5s;
}

.entry-menu.hidden {
    opacity: 0;
    transform: scale(0.9);
    pointer-events: none;
}

.menu-content {
    text-align: center;
    animation: menuFadeIn 1s ease-out;
    position: relative;
}

@keyframes menuFadeIn {
    from {
        opacity: 0;
        transform: translateY(-30px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.menu-title {
    font-size: 4rem;
    color: #fff;
    text-shadow: 
        0 0 10px rgba(255, 255, 255, 0.8),
        0 0 20px rgba(255, 215, 0, 0.6),
        0 0 30px rgba(255, 215, 0, 0.4),
        3px 3px 6px rgba(0, 0, 0, 0.5);
    margin-bottom: 20px;
    animation: titlePulse 2s ease-in-out infinite;
}

@keyframes titlePulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.05); }
}

.menu-subtitle {
    font-size: 1.5rem;
    color: #ffe5e5;
    margin-bottom: 40px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.menu-icons {
    display: flex;
    justify-content: center;
    gap: 30px;
    font-size: 3rem;
    margin-bottom: 50px;
}

.bounce {
    display: inline-block;
    animation: bounce 1s ease-in-out infinite;
}

.delay-1 { animation-delay: 0.1s; }
.delay-2 { animation-delay: 0.2s; }
.delay-3 { animation-delay: 0.3s; }
.delay-4 { animation-delay: 0.4s; }

@keyframes bounce {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-20px); }
}

.menu-buttons {
    display: flex;
    flex-direction: column;
    gap: 20px;
    margin-bottom: 30px;
}

.menu-btn {
    background: linear-gradient(135deg, #fff 0%, #f0f0f0 100%);
    border: 4px solid #d4af37;
    padding: 20px 40px;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.3s;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 15px;
    min-width: 350px;
}

.menu-btn:hover {
    transform: translateY(-5px) scale(1.05);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.5);
    border-color: #ffd700;
}

.easy-btn:hover { background: linear-gradient(135deg, #90EE90 0%, #7FFF7F 100%); }
.medium-btn:hover { background: linear-gradient(135deg, #FFD700 0%, #FFC700 100%); }
.hard-btn:hover { background: linear-gradient(135deg, #FF6B6B 0%, #FF5252 100%); }

.btn-icon {
    font-size: 2rem;
}

.btn-text {
    font-size: 1.5rem;
    font-weight: bold;
    color: #2c3e50;
}

.btn-detail {
    font-size: 0.9rem;
    color: #555;
}

.music-control {
    margin-top: 20px;
}

.music-btn {
    background: rgba(255, 255, 255, 0.2);
    border: 2px solid rgba(255, 255, 255, 0.5);
    color: white;
    padding: 12px 30px;
    border-radius: 30px;
    cursor: pointer;
    font-size: 1.1rem;
    transition: all 0.3s;
    backdrop-filter: blur(10px);
}

.music-btn:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: scale(1.1);
}

.music-btn.playing {
    background: rgba(76, 175, 80, 0.5);
    border-color: #4CAF50;
    animation: musicPulse 1s ease-in-out infinite;
}

@keyframes musicPulse {
    0%, 100% { box-shadow: 0 0 10px rgba(76, 175, 80, 0.5); }
    50% { box-shadow: 0 0 20px rgba(76, 175, 80, 0.8); }
}

/* Snowfall Animation */
.snow-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    overflow: hidden;
}

.snowflake {
    position: absolute;
    top: -50px;
    color: white;
    font-size: 1.5rem;
    opacity: 0.8;
    animation: fall linear infinite;
}

.snowflake:nth-child(1) { left: 10%; animation-duration: 8s; animation-delay: 0s; }
.snowflake:nth-child(2) { left: 20%; animation-duration: 10s; animation-delay: 1s; }
.snowflake:nth-child(3) { left: 30%; animation-duration: 7s; animation-delay: 2s; }
.snowflake:nth-child(4) { left: 40%; animation-duration: 9s; animation-delay: 0.5s; }
.snowflake:nth-child(5) { left: 50%; animation-duration: 11s; animation-delay: 1.5s; }
.snowflake:nth-child(6) { left: 60%; animation-duration: 8.5s; animation-delay: 0.8s; }
.snowflake:nth-child(7) { left: 70%; animation-duration: 9.5s; animation-delay: 1.2s; }
.snowflake:nth-child(8) { left: 80%; animation-duration: 10.5s; animation-delay: 0.3s; }
.snowflake:nth-child(9) { left: 90%; animation-duration: 7.5s; animation-delay: 1.8s; }
.snowflake:nth-child(10) { left: 95%; animation-duration: 9.8s; animation-delay: 0.6s; }

@keyframes fall {
    to {
        transform: translateY(100vh) rotate(360deg);
    }
}

/* Game Container */
.container {
    max-width: 900px;
    width: 100%;
    text-align: center;
    transition: opacity 0.5s;
}

.container.hidden {
    display: none;
}

.header-controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.back-btn {
    background: rgba(255, 255, 255, 0.2);
    border: 2px solid rgba(255, 255, 255, 0.5);
    color: white;
    padding: 10px 20px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 1rem;
    transition: all 0.3s;
    backdrop-filter: blur(10px);
}

.back-btn:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: translateX(-5px);
}

.music-btn-small {
    background: rgba(255, 255, 255, 0.2);
    border: 2px solid rgba(255, 255, 255, 0.5);
    color: white;
    padding: 10px 15px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 1rem;
    transition: all 0.3s;
    backdrop-filter: blur(10px);
}

.music-btn-small:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: scale(1.1);
}

.music-btn-small.playing {
    background: rgba(76, 175, 80, 0.5);
    border-color: #4CAF50;
}

h1 {
    color: #fff;
    font-size: 2.5rem;
    text-shadow: 
        0 0 10px rgba(255, 255, 255, 0.6),
        0 0 20px rgba(255, 215, 0, 0.4),
        2px 2px 4px rgba(0, 0, 0, 0.3);
    flex: 1;
}

.scoreboard {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 15px;
    padding: 20px;
    margin-bottom: 30px;
    display: flex;
    justify-content: space-around;
    align-items: center;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.score-item {
    font-size: 1.2rem;
    color: #165a4c;
    font-weight: bold;
}

.score-item .label {
    color: #0d3d33;
}

.score-item .value {
    color: #d32f2f;
    font-size: 1.5rem;
    margin: 0 5px;
}

.game-board {
    display: grid;
    gap: 15px;
    margin-bottom: 30px;
    padding: 20px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 15px;
    backdrop-filter: blur(10px);
}

.game-board.easy { grid-template-columns: repeat(4, 1fr); }
.game-board.medium { grid-template-columns: repeat(5, 1fr); }
.game-board.hard { grid-template-columns: repeat(6, 1fr); }

.card {
    aspect-ratio: 1;
    background: linear-gradient(135deg, #fff 0%, #f5f5f5 100%);
    border-radius: 10px;
    cursor: pointer;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.card:hover:not(.flipped):not(.matched) {
    transform: scale(1.05);
}

.card.flipped {
    transform: rotateY(180deg);
}

.card.matched {
    opacity: 1;
    cursor: default;
    animation: matchedPulse 0.5s;
}

.card.matched .card-face {
    background: linear-gradient(135deg, #7CFC00 0%, #32CD32 100%);
    color: #0d3318;
    box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.15);
}

@keyframes matchedPulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.1); }
}

.card-face,
.card-back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 3rem;
    border-radius: 10px;
}

.card-back {
    background: linear-gradient(135deg, #d32f2f 0%, #b71c1c 100%);
    color: white;
    box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.2);
}

.card-face {
    background: linear-gradient(135deg, #fff 0%, #f5f5f5 100%);
    transform: rotateY(180deg);
}

.restart-btn {
    background: linear-gradient(135deg, #d32f2f 0%, #b71c1c 100%);
    color: white;
    border: none;
    padding: 15px 40px;
    font-size: 1.2rem;
    border-radius: 30px;
    cursor: pointer;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    transition: transform 0.2s, box-shadow 0.2s;
    font-weight: bold;
}

.restart-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4);
}

.restart-btn:active {
    transform: translateY(0);
}

.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    animation: fadeIn 0.3s;
}

.modal.hidden {
    display: none;
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

.confetti {
    position: absolute;
    top: -20px;
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-around;
    font-size: 2rem;
    animation: confettiFall 3s ease-out infinite;
}

.confetti span {
    animation: confettiRotate 2s linear infinite;
}

@keyframes confettiFall {
    0% { transform: translateY(0); opacity: 1; }
    100% { transform: translateY(300px); opacity: 0; }
}

@keyframes confettiRotate {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

.modal-content {
    background: linear-gradient(135deg, #fff 0%, #f5f5f5 100%);
    padding: 40px;
    border-radius: 20px;
    text-align: center;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
    animation: slideIn 0.3s;
    max-width: 400px;
    width: 90%;
    position: relative;
    overflow: hidden;
}

@keyframes slideIn {
    from {
        transform: translateY(-50px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

.modal-content h2 {
    color: #165a4c;
    font-size: 2rem;
    margin-bottom: 20px;
}

.modal-content p {
    color: #0d3d33;
    font-size: 1.2rem;
    margin-bottom: 15px;
}

.modal-stats {
    background: rgba(22, 90, 76, 0.1);
    padding: 20px;
    border-radius: 10px;
    margin: 20px 0;
}

.modal-stats p {
    margin: 10px 0;
    font-size: 1.3rem;
}

.highlight {
    color: #d32f2f;
    font-weight: bold;
    font-size: 1.5rem;
}

.modal-buttons {
    display: flex;
    gap: 10px;
    justify-content: center;
    flex-wrap: wrap;
}

.modal-btn {
    background: linear-gradient(135deg, #165a4c 0%, #0d3d33 100%);
    color: white;
    border: none;
    padding: 15px 30px;
    font-size: 1.1rem;
    border-radius: 30px;
    cursor: pointer;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    transition: transform 0.2s;
    font-weight: bold;
}

.modal-btn.secondary {
    background: linear-gradient(135deg, #757575 0%, #616161 100%);
}

.modal-btn:hover {
    transform: translateY(-2px);
}

/* Responsive design */
@media (max-width: 768px) {
    .menu-title {
        font-size: 2.5rem;
    }

    .menu-subtitle {
        font-size: 1.2rem;
    }

    .menu-btn {
        min-width: 280px;
        padding: 15px 30px;
    }

    .btn-text {
        font-size: 1.2rem;
    }

    h1 {
        font-size: 1.8rem;
    }

    .game-board {
        gap: 10px;
        padding: 15px;
    }

    .card-face,
    .card-back {
        font-size: 2rem;
    }

    .scoreboard {
        flex-direction: column;
        gap: 15px;
    }

    .header-controls {
        flex-direction: column;
        gap: 10px;
    }

    .restart-btn {
        padding: 12px 30px;
        font-size: 1rem;
    }
}

@media (max-width: 480px) {
    .menu-title {
        font-size: 2rem;
    }

    .menu-icons {
        font-size: 2rem;
        gap: 15px;
    }

    .menu-btn {
        min-width: 240px;
        padding: 12px 20px;
    }

    h1 {
        font-size: 1.5rem;
    }

    .game-board {
        gap: 8px;
        padding: 10px;
    }

    .card-face,
    .card-back {
        font-size: 1.5rem;
    }

    .score-item {
        font-size: 1rem;
    }

    .score-item .value {
        font-size: 1.2rem;
    }
}







<!DOCTYPE html>
<html lang="bg">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Christmas memory</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <!-- Entry Menu -->
    <div id="entry-menu" class="entry-menu">
        <div class="menu-content">
            <div class="snow-container">
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
                <div class="snowflake">*</div>
            </div>
            
            <h1 class="menu-title">Christmas matching</h1>
            <p class="menu-subtitle">Find all pairs!</p>
            
            <div class="menu-icons">
                <span class="bounce">*</span>
                <span class="bounce delay-1">*</span>
                <span class="bounce delay-2">*</span>
                <span class="bounce delay-3">*</span>
                <span class="bounce delay-4">*</span>
            </div>
            
            <div class="menu-buttons">
                <button id="start-easy" class="menu-btn easy-btn">
                    <span class="btn-icon">1</span>
                    <span class="btn-text">Easy</span>
                    <span class="btn-detail">4x4 (8 pairs)</span>
                </button>
                
                <button id="start-medium" class="menu-btn medium-btn">
                    <span class="btn-icon">2</span>
                    <span class="btn-text">Medium</span>
                    <span class="btn-detail">4x5 (10 pairs)</span>
                </button>
                
                <button id="start-hard" class="menu-btn hard-btn">
                    <span class="btn-icon">3</span>
                    <span class="btn-text">Hard</span>
                    <span class="btn-detail">6x6 (18 pairs)</span>
                </button>
            </div>
            
            <div class="music-control">
                <button id="music-toggle" class="music-btn">Music Off</button>
            </div>
        </div>
    </div>

    <!-- Game Container -->
    <div id="game-container" class="container hidden">
        <div class="header-controls">
            <button id="back-to-menu" class="back-btn">Back</button>
            <h1>Koledna pamet</h1>
            <button id="music-toggle-game" class="music-btn-small">Music Off</button>
        </div>
        
        <div class="scoreboard">
            <div class="score-item">
                <span class="label">Moves:</span>
                <span id="moves" class="value">0</span>
            </div>
            <div class="score-item">
                <span class="label">Time:</span>
                <span id="timer" class="value">0</span>
                <span class="label">sec</span>
            </div>
            <div class="score-item">
                <span class="label">Level:</span>
                <span id="difficulty-display" class="value">Easy</span>
            </div>
        </div>

        <div id="game-board" class="game-board"></div>

        <button id="restart-btn" class="restart-btn">New game</button>

        <div id="win-modal" class="modal hidden">
            <div class="modal-content">
                <h2>Bravo!</h2>
                <p>You finished the game!</p>
                <div class="modal-stats">
                    <p>Level: <span id="final-difficulty" class="highlight">Easy</span></p>
                    <p>Moves: <span id="final-moves" class="highlight">0</span></p>
                    <p>Time: <span id="final-time" class="highlight">0</span> sec</p>
                </div>
                <div class="modal-buttons">
                    <button id="modal-restart-btn" class="modal-btn">Play again</button>
                    <button id="modal-menu-btn" class="modal-btn secondary">Menu</button>
                </div>
            </div>
        </div>
    </div>

    <script src="music-generator.js"></script>
    <script src="script.js"></script>
</body>
</html>






using System.Net;
using System.Text;

Console.WriteLine("🎄 Стартиране на Коледна памет игра сървър...");
Console.WriteLine("Отворете браузър и посетете: http://localhost:8080");
Console.WriteLine("Натиснете Ctrl+C за да спрете сървъра.");

var server = new SimpleHttpServer(8080, Path.Combine(AppContext.BaseDirectory, "wwwroot"));
await server.StartAsync();

class SimpleHttpServer
{
    private readonly HttpListener _listener;
    private readonly string _rootPath;
    
    public SimpleHttpServer(int port, string rootPath)
    {
        _listener = new HttpListener();
        _listener.Prefixes.Add($"http://localhost:{port}/");
        _rootPath = rootPath;
    }
    
    public async Task StartAsync()
    {
        _listener.Start();
        Console.WriteLine($"Сървърът работи на http://localhost:8080");
        
        while (true)
        {
            var context = await _listener.GetContextAsync();
            _ = HandleRequestAsync(context);
        }
    }
    
    private async Task HandleRequestAsync(HttpListenerContext context)
    {
        try
        {
            var request = context.Request;
            var response = context.Response;
            
            string requestedPath = request.Url?.LocalPath ?? "/";
            if (requestedPath == "/")
                requestedPath = "/index.html";
            
            string filePath = Path.Combine(_rootPath, requestedPath.TrimStart('/'));
            
            if (File.Exists(filePath))
            {
                byte[] buffer = await File.ReadAllBytesAsync(filePath);
                
                response.ContentType = GetContentType(filePath);
                response.ContentLength64 = buffer.Length;
                response.StatusCode = 200;
                
                await response.OutputStream.WriteAsync(buffer);
            }
            else
            {
                response.StatusCode = 404;
                byte[] buffer = Encoding.UTF8.GetBytes("404 - Файлът не е намерен");
                await response.OutputStream.WriteAsync(buffer);
            }
            
            response.OutputStream.Close();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Грешка: {ex.Message}");
        }
    }
    
    private string GetContentType(string filePath)
    {
        string extension = Path.GetExtension(filePath).ToLower();
        return extension switch
        {
            ".html" => "text/html; charset=utf-8",
            ".css" => "text/css; charset=utf-8",
            ".js" => "application/javascript; charset=utf-8",
            ".png" => "image/png",
            ".jpg" or ".jpeg" => "image/jpeg",
            ".gif" => "image/gif",
            ".svg" => "image/svg+xml",
            ".ico" => "image/x-icon",
            _ => "application/octet-stream"
        };
    }
}
