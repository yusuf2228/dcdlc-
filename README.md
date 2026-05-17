# dcdlc-
A DC Comics daily guessing game
import express from 'express';
import cors from 'cors';
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

const DC_CHARACTERS = [
  {
    id: 1,
    name: 'Superman',
    realName: 'Clark Kent',
    alias: 'Kal-El',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Super Strength', 'Flight', 'Heat Vision', 'X-Ray Vision'],
    firstAppearance: '1938',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Superman'
  },
  {
    id: 2,
    name: 'Batman',
    realName: 'Bruce Wayne',
    alias: 'The Dark Knight',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Intelligence', 'Martial Arts', 'Wealth'],
    firstAppearance: '1939',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Batman'
  },
  {
    id: 3,
    name: 'Wonder Woman',
    realName: 'Diana Prince',
    alias: 'Diana of Themyscira',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Super Strength', 'Immortality', 'Lasso of Truth', 'Combat Skills'],
    firstAppearance: '1941',
    gender: 'Female',
    imageUrl: 'https://via.placeholder.com/300?text=Wonder+Woman'
  },
  {
    id: 4,
    name: 'The Flash',
    realName: 'Barry Allen',
    alias: 'The Scarlet Speedster',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Super Speed', 'Speed Force', 'Vibration'],
    firstAppearance: '1956',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=The+Flash'
  },
  {
    id: 5,
    name: 'Aquaman',
    realName: 'Arthur Curry',
    alias: 'Orin',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Aquatic Communication', 'Super Strength', 'Durability'],
    firstAppearance: '1941',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Aquaman'
  },
  {
    id: 6,
    name: 'Green Lantern',
    realName: 'Hal Jordan',
    alias: 'Space Cop',
    universe: 'DC Earth-0',
    affiliation: 'Justice League',
    powers: ['Ring Constructs', 'Flight', 'Energy Projection'],
    firstAppearance: '1959',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Green+Lantern'
  },
  {
    id: 7,
    name: 'Joker',
    realName: 'Unknown',
    alias: 'The Clown Prince',
    universe: 'DC Earth-0',
    affiliation: 'Villain',
    powers: ['Chemistry', 'Unpredictability'],
    firstAppearance: '1940',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Joker'
  },
  {
    id: 8,
    name: 'Harley Quinn',
    realName: 'Harleen Quinzel',
    alias: 'Harlequin',
    universe: 'DC Earth-0',
    affiliation: 'Anti-Hero',
    powers: ['Gymnastics', 'Unpredictability', 'Baseball'],
    firstAppearance: '1992',
    gender: 'Female',
    imageUrl: 'https://via.placeholder.com/300?text=Harley+Quinn'
  },
  {
    id: 9,
    name: 'Lex Luthor',
    realName: 'Alexander Luthor',
    alias: 'The Man of Steel\'s Nemesis',
    universe: 'DC Earth-0',
    affiliation: 'Villain',
    powers: ['Intelligence', 'Wealth', 'Technology'],
    firstAppearance: '1940',
    gender: 'Male',
    imageUrl: 'https://via.placeholder.com/300?text=Lex+Luthor'
  },
  {
    id: 10,
    name: 'Black Widow',
    realName: 'Pamela Isley',
    alias: 'Poison Ivy',
    universe: 'DC Earth-0',
    affiliation: 'Anti-Hero',
    powers: ['Plant Control', 'Toxins', 'Seduction'],
    firstAppearance: '1966',
    gender: 'Female',
    imageUrl: 'https://via.placeholder.com/300?text=Poison+Ivy'
  }
];

function getDailyCharacter() {
  const today = new Date();
  const date = today.getFullYear() + '-' + (today.getMonth() + 1) + '-' + today.getDate();
  const seed = date.split('').reduce((a, b) => a + b.charCodeAt(0), 0);
  const index = seed % DC_CHARACTERS.length;
  return DC_CHARACTERS[index];
}

function calculateDifferences(guessChar, targetChar) {
  const differences = {};
  
  differences.name = guessChar.name === targetChar.name ? 'correct' : 'incorrect';
  differences.realName = guessChar.realName === targetChar.realName ? 'correct' : 'incorrect';
  differences.gender = guessChar.gender === targetChar.gender ? 'correct' : 'incorrect';
  differences.affiliation = guessChar.affiliation === targetChar.affiliation ? 'correct' : 'incorrect';
  differences.firstAppearance = guessChar.firstAppearance === targetChar.firstAppearance ? 'correct' : guessChar.firstAppearance < targetChar.firstAppearance ? 'earlier' : 'later';
  
  const guessPowers = new Set(guessChar.powers);
  const targetPowers = new Set(targetChar.powers);
  const matches = [...guessPowers].filter(p => targetPowers.has(p)).length;
  differences.powers = matches > 0 ? `${matches}/${guessPowers.size}` : 'none';
  
  return differences;
}

app.get('/api/daily-character', (req, res) => {
  const dailyChar = getDailyCharacter();
  const { imageUrl, ...charWithoutImage } = dailyChar;
  res.json(charWithoutImage);
});

app.get('/api/search', (req, res) => {
  const query = req.query.q?.toLowerCase() || '';
  if (!query) {
    res.json([]);
    return;
  }
  const results = DC_CHARACTERS.filter(char => 
    char.name.toLowerCase().includes(query) ||
    char.realName.toLowerCase().includes(query) ||
    char.alias.toLowerCase().includes(query)
  ).map(char => ({
    id: char.id,
    name: char.name,
    realName: char.realName
  }));
  res.json(results);
});

app.post('/api/guess', (req, res) => {
  const { characterId } = req.body;
  const guessChar = DC_CHARACTERS.find(c => c.id === characterId);
  const targetChar = getDailyCharacter();
  
  if (!guessChar) {
    return res.status(404).json({ error: 'Character not found' });
  }
  
  const isCorrect = guessChar.id === targetChar.id;
  const differences = calculateDifferences(guessChar, targetChar);
  
  res.json({
    isCorrect,
    differences,
    targetCharacter: isCorrect ? targetChar : null
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`DCdle server running on http://localhost:${PORT}`);
});let guessHistory = [];
let gameWon = false;
let dailyCharacterId = null;

const searchInput = document.getElementById('searchInput');
const suggestionsDiv = document.getElementById('suggestions');
const guessesContainer = document.getElementById('guesses');
const messageDiv = document.getElementById('message');
const statsDiv = document.getElementById('stats');
const shareBtn = document.getElementById('shareBtn');

// Search for characters
searchInput.addEventListener('input', async (e) => {
    const query = e.target.value.trim();
    
    if (!query) {
        suggestionsDiv.classList.remove('active');
        return;
    }

    try {
        const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
        const results = await response.json();
        
        if (results.length === 0) {
            suggestionsDiv.classList.remove('active');
            return;
        }

        suggestionsDiv.innerHTML = results.map(char => `
            <div class="suggestion-item" onclick="selectCharacter(${char.id}, '${char.name}')">
                <strong>${char.name}</strong>
                <br>
                <small>${char.realName}</small>
            </div>
        `).join('');
        
        suggestionsDiv.classList.add('active');
    } catch (error) {
        console.error('Search error:', error);
    }
});

// Close suggestions when clicking outside
document.addEventListener('click', (e) => {
    if (e.target !== searchInput) {
        suggestionsDiv.classList.remove('active');
    }
});

function selectCharacter(characterId, characterName) {
    if (gameWon) return;

    searchInput.value = '';
    suggestionsDiv.classList.remove('active');
    
    makeGuess(characterId);
}

async function makeGuess(characterId) {
    try {
        const response = await fetch('/api/guess', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ characterId })
        });

        const data = await response.json();

        if (!data.targetCharacter && !data.differences) {
            messageDiv.textContent = 'Character not found!';
            messageDiv.className = 'message error';
            return;
        }

        guessHistory.push({
            characterId,
            differences: data.differences
        });

        displayGuess(data);

        if (data.isCorrect) {
            gameWon = true;
            messageDiv.textContent = '🎉 Correct! You got it!';
            messageDiv.className = 'message success';
            statsDiv.style.display = 'block';
            searchInput.disabled = true;
        } else {
            messageDiv.textContent = `${guessHistory.length} guess${guessHistory.length > 1 ? 'es' : ''} so far...`;
            messageDiv.className = 'message';
        }
    } catch (error) {
        console.error('Guess error:', error);
        messageDiv.textContent = 'Error making guess. Try again!';
        messageDiv.className = 'message error';
    }
}

function displayGuess(guessData) {
    const { differences } = guessData;
    const guess = guessHistory[guessHistory.length - 1];

    const guessCard = document.createElement('div');
    guessCard.className = 'guess-card';
    
    let html = `<div class="guess-header">Guess #${guessHistory.length}</div><div class="guess-details">`;

    const detailsArray = [
        { label: 'Name', value: differences.name },
        { label: 'Real Name', value: differences.realName },
        { label: 'Gender', value: differences.gender },
        { label: 'Affiliation', value: differences.affiliation },
        { label: 'First Appearance', value: differences.firstAppearance },
        { label: 'Powers', value: differences.powers }
    ];

    detailsArray.forEach(detail => {
        html += `
            <div class="detail-row">
                <span class="detail-label">${detail.label}</span>
                <span class="detail-value ${detail.value.includes('none') ? 'incorrect' : detail.value === 'correct' ? 'correct' : detail.value === 'earlier' ? 'earlier' : detail.value === 'later' ? 'later' : ''}"><strong>${detail.value}</strong></span>
            </div>
        `;
    });

    html += '</div>';
    guessCard.innerHTML = html;
    guessesContainer.prepend(guessCard);
}

shareBtn.addEventListener('click', () => {
    const emoji = gameWon ? '🎉' : '❌';
    const text = `I solved DCdle in ${guessHistory.length} guess${guessHistory.length > 1 ? 'es' : ''}! ${emoji}\n\nCan you guess the DC character of the day?\nhttps://dcdlc.vercel.app`;
    
    if (navigator.share) {
        navigator.share({
            title: 'DCdle',
            text: text
        });
    } else {
        navigator.clipboard.writeText(text);
        alert('Result copied to clipboard!');
    }
});

// Load game on start
window.addEventListener('load', () => {
    // Check if there are any guesses saved in local storage for today
    const today = new Date().toDateString();
    const saved = localStorage.getItem('dcdle_' + today);
    
    if (saved) {
        const data = JSON.parse(saved);
        guessHistory = data.guesses;
        gameWon = data.won;
        
        if (gameWon) {
            statsDiv.style.display = 'block';
            searchInput.disabled = true;
        }
        
        guessHistory.forEach(guess => {
            const card = document.createElement('div');
            card.className = 'guess-card';
            
            let html = `<div class="guess-header">Guess #${guessHistory.indexOf(guess) + 1}</div><div class="guess-details">`;
            
            const detailsArray = [
                { label: 'Name', value: guess.differences.name },
                { label: 'Real Name', value: guess.differences.realName },
                { label: 'Gender', value: guess.differences.gender },
                { label: 'Affiliation', value: guess.differences.affiliation },
                { label: 'First Appearance', value: guess.differences.firstAppearance },
                { label: 'Powers', value: guess.differences.powers }
            ];

            detailsArray.forEach(detail => {
                html += `
                    <div class="detail-row">
                        <span class="detail-label">${detail.label}</span>
                        <span class="detail-value ${detail.value.includes('none') ? 'incorrect' : detail.value === 'correct' ? 'correct' : detail.value === 'earlier' ? 'earlier' : detail.value === 'later' ? 'later' : ''}"><strong>${detail.value}</strong></span>
                    </div>
                `;
            });

            html += '</div>';
            card.innerHTML = html;
            guessesContainer.appendChild(card);
        });
    }
});

// Save game state when guess is made
function saveGameState() {
    const today = new Date().toDateString();
    localStorage.setItem('dcdle_' + today, JSON.stringify({
        guesses: guessHistory,
        won: gameWon
    }));
}

// Update save on each guess
const originalMakeGuess = makeGuess;
makeGuess = async function(characterId) {
    await originalMakeGuess(characterId);
    saveGameState();
};<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DCdle - DC Comics Daily Guess</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🦸 DCdle 🦸</h1>
            <p>Guess the DC Comics character of the day!</p>
        </header>

        <main>
            <div class="game-section">
                <div class="search-box">
                    <input 
                        type="text" 
                        id="searchInput" 
                        placeholder="Search DC characters..."
                        autocomplete="off"
                    >
                    <div id="suggestions" class="suggestions"></div>
                </div>

                <div id="guesses" class="guesses-container"></div>

                <div id="message" class="message"></div>
            </div>

            <div id="stats" class="stats" style="display: none;">
                <h2>🎉 You Won!</h2>
                <button id="shareBtn" class="share-btn">Share Result</button>
            </div>
        </main>

        <footer>
            <p>A fan-made game inspired by Marveldle | Not affiliated with DC Comics</p>
        </footer>
    </div>

    <script src="app.js"></script>
</body>
</html>* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    color: #fff;
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 600px;
    margin: 0 auto;
}

header {
    text-align: center;
    margin-bottom: 40px;
    padding-top: 20px;
}

header h1 {
    font-size: 3em;
    margin-bottom: 10px;
    background: linear-gradient(135deg, #dc143c, #ffd700);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
}

header p {
    color: #aaa;
    font-size: 1.1em;
}

main {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
    padding: 30px;
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 255, 255, 0.1);
}

.search-box {
    position: relative;
    margin-bottom: 30px;
}

#searchInput {
    width: 100%;
    padding: 15px;
    font-size: 1em;
    border: 2px solid #dc143c;
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
    transition: all 0.3s ease;
}

#searchInput:focus {
    outline: none;
    border-color: #ffd700;
    background: rgba(255, 255, 255, 0.15);
    box-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
}

#searchInput::placeholder {
    color: #aaa;
}

.suggestions {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: rgba(26, 26, 46, 0.95);
    border: 1px solid #dc143c;
    border-top: none;
    border-radius: 0 0 8px 8px;
    max-height: 300px;
    overflow-y: auto;
    display: none;
    z-index: 10;
}

.suggestions.active {
    display: block;
}

.suggestion-item {
    padding: 12px 15px;
    cursor: pointer;
    transition: background 0.2s;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.suggestion-item:hover {
    background: rgba(220, 20, 60, 0.3);
}

.suggestion-item strong {
    color: #ffd700;
}

.guesses-container {
    margin-bottom: 20px;
}

.guess-card {
    background: rgba(255, 255, 255, 0.08);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    padding: 15px;
    margin-bottom: 15px;
    animation: slideIn 0.3s ease;
}

@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.guess-header {
    font-size: 1.2em;
    font-weight: bold;
    margin-bottom: 10px;
    color: #ffd700;
}

.guess-details {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    font-size: 0.9em;
}

.detail-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 5px;
}

.detail-label {
    color: #aaa;
    font-weight: bold;
}

.detail-value {
    margin-left: 10px;
}

.correct {
    color: #00ff00;
    font-weight: bold;
}

.incorrect {
    color: #ff4444;
}

.earlier {
    color: #87ceeb;
}

.later {
    color: #dda0dd;
}

.message {
    text-align: center;
    font-size: 1.2em;
    margin: 20px 0;
    min-height: 30px;
}

.message.error {
    color: #ff4444;
}

.message.success {
    color: #00ff00;
    font-weight: bold;
}

.stats {
    text-align: center;
    margin-top: 30px;
    padding-top: 30px;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.stats h2 {
    margin-bottom: 20px;
    font-size: 1.8em;
}

.share-btn {
    background: linear-gradient(135deg, #dc143c, #ffd700);
    color: white;
    border: none;
    padding: 12px 30px;
    font-size: 1em;
    border-radius: 8px;
    cursor: pointer;
    transition: transform 0.2s;
}

.share-btn:hover {
    transform: scale(1.05);
}

footer {
    text-align: center;
    margin-top: 40px;
    color: #666;
    font-size: 0.9em;
}

@media (max-width: 600px) {
    header h1 {
        font-size: 2em;
    }

    main {
        padding: 20px;
    }

    .guess-details {
        grid-template-columns: 1fr;
    }
}{
  "name": "dcdle",
  "version": "1.0.0",
  "description": "A DC Comics daily guessing game inspired by Marveldle",
  "main": "server.js",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  },
  "devDependencies": {}
}
