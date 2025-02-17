# warpixel
war pixel
﻿<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pixel Country War</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #1a1a1a;
            color: white;
        }
        #controls {
            margin-bottom: 20px;
            padding: 20px;
            background: #2a2a2a;
            border-radius: 10px;
        }
        .input-group {
            margin-bottom: 15px;
        }
        input[type="text"], input[type="number"] {
            padding: 8px;
            margin-right: 10px;
            border-radius: 5px;
            border: 1px solid #444;
            background: #333;
            color: white;
        }
        button {
            padding: 10px 20px;
            background: #007bff;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            background: #0056b3;
        }
        #pixelGrid {
            display: grid;
            grid-template-columns: repeat(30, 15px);
            gap: 2px;
            margin: 20px 0;
        }
        .pixel {
            width: 15px;
            height: 15px;
            position: relative;
            transition: background-color 0.5s;
            border-radius: 3px;
        }
        .pixel.capital {
            background-color: black !important; /* Черная столица */
            border: 1px solid white; /* Белая рамка */
        }
        .pixel-label {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 8px;
            color: white;
            text-shadow: 1px 1px 2px black;
            pointer-events: none;
        }
        @keyframes battle {
            0% { transform: scale(1); }
            50% { transform: scale(1.3); }
            100% { transform: scale(1); }
        }
        .battle {
            animation: battle 0.5s ease-out;
        }
        #victory-message {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.9);
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            display: none;
        }
        #statsPanel {
            margin-top: 20px;
            padding: 20px;
            background: #2a2a2a;
            border-radius: 10px;
        }
        .country-stat {
            margin-bottom: 10px;
            padding: 10px;
            border-left: 5px solid;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="controls">
        <div class="input-group">
            <input type="text" id="countryName" placeholder="Country name">
            <input type="color" id="countryColor" value="#ff0000">
            <input type="number" id="armySize" placeholder="Army size" min="1" max="1000">
            <button id="createCountryButton">Create Country</button>
            <button id="randomArmyButton">🎲 Random Army</button>
        </div>
        <button id="startWarButton">Start World War!</button>
    </div>

    <div id="pixelGrid"></div>
    <div id="statsPanel">
        <h3>Army Strength</h3>
        <div id="countryStats"></div>
    </div>
    <div id="victory-message"></div>

    <script>
        const GRID_SIZE = 30;
        const WIN_PERCENTAGE = 60;
        const pixels = [];
        let countries = [];
        let warInterval = null;
        let gameActive = false;

        // Initialize grid
        function initializeGrid() {
            const grid = document.getElementById('pixelGrid');
            grid.innerHTML = '';
            for (let i = 0; i < GRID_SIZE * GRID_SIZE; i++) {
                const pixel = document.createElement('div');
                pixel.className = 'pixel';
                pixel.style.backgroundColor = '#333';
                
                const label = document.createElement('div');
                label.className = 'pixel-label';
                pixel.appendChild(label);
                
                grid.appendChild(pixel);
                pixels.push({
                    element: pixel,
                    label: label,
                    owner: null,
                    army: 0
                });
            }
        }

        // Create new country
        function createCountry() {
            const name = document.getElementById('countryName').value.trim();
            const color = document.getElementById('countryColor').value;
            const armySize = parseInt(document.getElementById('armySize').value);
            
            if (!name) {
                alert('Please enter a country name!');
                return;
            }
            if (isNaN(armySize) || armySize < 1) {
                alert('Please enter a valid army size (min: 1)!');
                return;
            }

            let startIndex;
            do {
                startIndex = Math.floor(Math.random() * GRID_SIZE * GRID_SIZE);
            } while (pixels[startIndex].owner !== null);

            const newCountry = {
                name,
                color,
                territory: new Set([startIndex]),
                army: armySize,
                capital: startIndex
            };

            countries.push(newCountry);
            updatePixel(startIndex, newCountry);
            document.getElementById('countryName').value = '';
            document.getElementById('armySize').value = '';
            updateStatsPanel();
        }

        // Update pixel appearance
        function updatePixel(index, country) {
            const pixel = pixels[index].element;
            pixel.style.backgroundColor = country.color;
            pixel.classList.toggle('capital', index === country.capital); // Черная столица
            pixels[index].label.textContent = country.name;
            pixels[index].owner = country;
            pixels[index].army = country.army;
        }

        // Random army size
        document.getElementById('randomArmyButton').addEventListener('click', () => {
            document.getElementById('armySize').value = Math.floor(Math.random() * 1000) + 1;
        });

        // Update stats panel
        function updateStatsPanel() {
            const statsContainer = document.getElementById('countryStats');
            statsContainer.innerHTML = countries.map(country => `
                <div class="country-stat" style="border-left-color: ${country.color}">
                    ${country.name}: ${Math.round(country.army)}
                </div>
            `).join('');
        }

        // Check for victory
        function checkVictory() {
            const remainingCountries = countries.filter(c => c.territory.size > 0);
            if (remainingCountries.length === 1) {
                declareVictory(remainingCountries[0]);
                return true;
            }

            for (const country of countries) {
                const percentage = (country.territory.size / (GRID_SIZE * GRID_SIZE)) * 100;
                if (percentage >= WIN_PERCENTAGE) {
                    declareVictory(country);
                    return true;
                }
            }
            return false;
        }

        // Declare victory
        function declareVictory(winner) {
            gameActive = false;
            clearInterval(warInterval);
            document.getElementById('startWarButton').textContent = 'Start World War!';
            const victoryMessage = document.getElementById('victory-message');
            victoryMessage.innerHTML = `
                <h2>${winner.name} WINS!</h2>
                <p>Captured ${Math.round((winner.territory.size / (GRID_SIZE * GRID_SIZE)) * 100)}% of territory</p>
                <button onclick="location.reload()">New Game</button>
            `;
            victoryMessage.style.display = 'block';
        }

        // Main battle logic
        function fight() {
            if (!gameActive) return;
            
            countries.forEach(country => {
                if (country.territory.size === 0) return;
                
                const borders = new Set();
                country.territory.forEach(index => {
                    const neighbors = getNeighbors(index);
                    neighbors.forEach(neighbor => {
                        if (pixels[neighbor].owner !== country) {
                            borders.add(neighbor);
                        }
                    });
                });

                if (borders.size > 0) {
                    const target = Array.from(borders)[Math.floor(Math.random() * borders.size)];
                    const defender = pixels[target].owner;
                    
                    if (defender) {
                        const attackPower = country.army * (1 + Math.random());
                        const defensePower = defender.army * (1 + Math.random());
                        
                        if (attackPower > defensePower) {
                            defender.territory.delete(target);
                            country.territory.add(target);
                            country.army -= Math.floor(attackPower * 0.1);
                            defender.army -= Math.floor(defensePower * 0.1);
                            updatePixel(target, country);
                            pixels[target].element.classList.add('battle');
                            setTimeout(() => {
                                pixels[target].element.classList.remove('battle');
                            }, 500);

                            if (target === defender.capital) {
                                defender.territory.clear();
                            }
                        }
                    } else {
                        country.territory.add(target);
                        updatePixel(target, country);
                    }
                }

                // Increase army if not fighting
                if (borders.size === 0) {
                    country.army += Math.floor(country.territory.size * 0.1);
                }
            });

            updateStatsPanel();
            if (checkVictory()) {
                return;
            }
        }

        // Get neighboring pixels
        function getNeighbors(index) {
            const neighbors = [];
            const x = index % GRID_SIZE;
            const y = Math.floor(index / GRID_SIZE);

            if (x > 0) neighbors.push(index - 1);
            if (x < GRID_SIZE - 1) neighbors.push(index + 1);
            if (y > 0) neighbors.push(index - GRID_SIZE);
            if (y < GRID_SIZE - 1) neighbors.push(index + GRID_SIZE);

            return neighbors;
        }

        // Start/pause war
        function startWar() {
            if (countries.length < 2) {
                alert('Create at least 2 countries to start war!');
                return;
            }

            const startWarButton = document.getElementById('startWarButton');
            if (warInterval) {
                gameActive = false;
                clearInterval(warInterval);
                warInterval = null;
                startWarButton.textContent = 'Start World War!';
            } else {
                gameActive = true;
                warInterval = setInterval(fight, 100);
                startWarButton.textContent = 'Stop War';
                document.getElementById('victory-message').style.display = 'none';
            }
        }

        // Initialize the grid when page loads
        initializeGrid();
        document.getElementById('createCountryButton').addEventListener('click', createCountry);
        document.getElementById('startWarButton').addEventListener('click', startWar);
    </script>
</body>
</html>
