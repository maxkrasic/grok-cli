# Proving Grounds: Complete Game Design Document

## 1. Executive Summary

**Core Hook**: Fantasy Football meets Combat Sports - manage fighters, not teams.

Proving Grounds is a fantasy-sports-style combat simulator where players manage individual fighters across seasonal leagues. Each week, players allocate training hours (H) to improve stats, set fight tactics via sliders, then watch AI-simulated battles unfold. Success comes from strategic resource management, opponent analysis, and tactical adaptation.

**Unique Value**: Unlike fantasy sports that rely on real-world performance, Proving Grounds uses deterministic AI simulation with controlled variance. Players directly influence outcomes through training decisions and fight tactics. No pay-to-win mechanics - only cosmetic purchases with Cash (C) currency.

**MVP**: Single-file HTML prototype with fighter creation, training allocation, slider-based fight interface, text-based combat simulation, and league standings. Persistent localStorage saves.

**Long-term Vision**: Multi-fighter management, gym partnerships, live spectating, global tournaments, and expanded fighting styles. Core loop remains: Train → Lock → Fight → Progress.

**Target**: 6-12 player leagues, weekly/bi-weekly pacing, seasonal progression with stat carryover normalization. Soft counters between fighting styles prevent dominant strategies while rewarding specialization and adaptation.

---

## 2. Gameplay Design Document (GDD)

### Game Loop
1. **Train Phase** (5 days): Allocate training hours (H) across stats
2. **Lock Phase** (1 day): Set fight tactics via sliders, review opponent
3. **Fight Phase** (1 day): AI simulation executes, results broadcast
4. **Progress Phase**: Update standings, advance to next week

### Player Progression & Stat System

**Core Stats** (0-100 scale, soft cap at 80):
- **Strength**: Damage output, takedown power
- **Speed**: Strike accuracy, movement, reaction time  
- **Skill**: Technique efficiency, combo potential
- **Endurance**: Stamina pool, recovery rate
- **Chin**: Damage resistance, knockout threshold
- **Fight IQ**: Tactical adaptation, counter-recognition

**Aging System**: -1 Speed OR Endurance per season (player choice)

### Training Economy (Hours = H)

**Weekly H Allocation**: 40 hours base + 10 bonus hours
- Base training: Linear gains up to stat 60
- Advanced training: Exponential cost curve 60-80
- Elite training: Extreme cost curve 80-100

**Training Efficiency Formula**:
```
Cost = Base_Cost × (1.5 ^ (Current_Stat - 50) / 10)
Gain = Hours_Spent / Cost
```

**Trade-offs**: Heavy strength training reduces next fight's stamina efficiency by 10%. Speed training reduces power by 5%. Balanced training has no penalties.

### Matchmaking Systems

**Round Robin**: Everyone fights everyone once
**Swiss System**: Pair similar records, 5-7 rounds
**Ranked Ladder**: ELO-based matching, climb/fall

### Fight Simulation Logic

**Pre-Fight Inputs**:
- Distance Preference (0-100): Close/Clinch vs Long Range
- Tempo (0-100): Aggressive vs Patient
- Tactics (0-100): Technical vs Brawling
- Grapple Intent (0-100): Striking vs Wrestling/BJJ

**Simulation Core**:
- 3 rounds × 5 minutes = 15 combat phases
- Each phase: Calculate actions, resolve damage, update stamina
- Between rounds: "Corner cards" provide tactical adjustments

### Styles Matrix

| Style | Strength | Weakness | Bonus |
|-------|----------|----------|-------|
| Boxer | Speed +10, Skill +5 | Grapple Defense -10 | +15% standing damage |
| Wrestler | Strength +10, Endurance +5 | Speed -5 | +20% takedown success |
| BJJ | Skill +15 | Strength -5 | +25% submission damage |
| Kickboxer | Speed +5, Skill +5 | Chin -5 | +10% leg kick damage |
| Pressure | Endurance +10, Chin +5 | Fight IQ -5 | +15% combo damage |
| Counter | Fight IQ +15 | Endurance -5 | +20% counter-attack damage |

**Style Advantages** (soft counters):
- Boxer > Kickboxer (10% advantage)
- Wrestler > Boxer (15% advantage)  
- BJJ > Wrestler (10% advantage)
- Kickboxer > BJJ (15% advantage)
- Pressure > Counter (10% advantage)
- Counter > Pressure (15% advantage)

### Cosmetic Currency (Cash = C)

**Earning C**:
- Fight participation: 100C
- Win bonus: +50C
- Performance bonus: +25C (finish, comeback)
- Season placement: 200-500C based on rank

**Cosmetic Shop**:
- Fighter appearance: 200-500C
- Victory celebrations: 300C
- Corner team customization: 400C
- Arena preferences: 500C

### League Settings

**Seasonal Leagues** (8-12 weeks):
- Stat carryover with normalization
- Championship playoffs
- Aging effects apply

**Infinite Leagues**:
- Continuous play
- No aging
- Rolling standings

### Anti-Pay-to-Win Rules

1. No H purchases with real money
2. No stat boosts from microtransactions  
3. Cosmetics only affect appearance
4. All gameplay advantages earned through play
5. Training efficiency same for all players

### Catch-Up Mechanics

**New Player Protection**:
- +20% H efficiency for first 3 fights
- Matched against similar experience levels
- Bonus H for losses (10 extra hours)

**League Balance**:
- Stat normalization on league entry
- Performance-based H bonuses
- Underdog advantages in matchmaking

---

## 3. AI Fight Simulation Specification

### Input Parameters

**Fighter Stats** (0-100 each):
- Strength, Speed, Skill, Endurance, Chin, Fight IQ

**Tactical Sliders** (0-100 each):
- Distance: 0=Clinch/Close, 100=Long Range
- Tempo: 0=Patient, 100=Aggressive  
- Tactics: 0=Technical, 100=Brawling
- Grapple: 0=Striking Focus, 100=Wrestling/BJJ

**Style Modifiers**: Applied to base stats before simulation

### Core Simulation Engine

**Phase Structure**:
```
Fight = 3 Rounds
Round = 5 Combat Phases (1 minute each)
Phase = Action Selection → Resolution → State Update
```

**Action Selection Formula**:
```javascript
function selectAction(fighter, opponent, phase) {
    const distanceWeight = fighter.distance / 100;
    const tempoWeight = fighter.tempo / 100;
    const skillFactor = fighter.skill / 100;
    const iqFactor = fighter.fightIQ / 100;
    
    // Calculate action probabilities
    const strikeChance = (1 - fighter.grapple/100) * (0.6 + tempoWeight * 0.3);
    const grappleChance = (fighter.grapple/100) * (0.4 + skillFactor * 0.2);
    const defendChance = (1 - tempoWeight) * (0.3 + iqFactor * 0.2);
    
    return weightedRandom([
        {action: 'strike', weight: strikeChance},
        {action: 'grapple', weight: grappleChance}, 
        {action: 'defend', weight: defendChance}
    ]);
}
```

**Damage Calculation**:
```javascript
function calculateDamage(attacker, defender, actionType) {
    const baseDamage = attacker.strength * 0.3 + attacker.skill * 0.2;
    const accuracy = (attacker.speed - defender.speed + 50) / 100;
    const defense = defender.chin * 0.4 + defender.skill * 0.1;
    
    let damage = baseDamage * accuracy - defense;
    
    // Style bonuses
    damage *= getStyleBonus(attacker.style, defender.style, actionType);
    
    // Random variance ±15%
    damage *= (0.85 + Math.random() * 0.3);
    
    return Math.max(0, damage);
}
```

**Stamina System**:
```javascript
function updateStamina(fighter, action, phase) {
    const baseStaminaCost = {
        'strike': 8,
        'grapple': 12,
        'defend': 4
    };
    
    const cost = baseStaminaCost[action] * (1 - fighter.endurance/200);
    const recovery = fighter.endurance * 0.1;
    
    fighter.stamina = Math.max(0, fighter.stamina - cost + recovery);
    
    // Stamina affects all actions
    const staminaFactor = fighter.stamina / 100;
    fighter.currentSpeed *= staminaFactor;
    fighter.currentStrength *= staminaFactor;
}
```

### Between-Round Corner Cards

**Tactical Adjustments** (random selection):
- "Focus on leg kicks" (+10% leg damage, -5% head damage)
- "Pressure the pace" (+15% tempo, -10% defense)
- "Look for takedowns" (+20% grapple intent, -10% striking)
- "Stay patient" (-20% tempo, +15% counter chance)
- "Target the body" (+15% body damage, -10% head damage)

### Random Seed & Determinism

**Seed Generation**: `Math.seedrandom(fighter1.id + fighter2.id + fightDate)`
**Replay Support**: Same seed produces identical fight
**Variance Control**: ±15% damage, ±10% accuracy, ±5% stamina costs

---

## 4. Data Schema (JSON)

### Fighter Schema
```json
{
  "id": "fighter_uuid",
  "name": "Fighter Name",
  "style": "Boxer",
  "stats": {
    "strength": 65,
    "speed": 72,
    "skill": 68,
    "endurance": 70,
    "chin": 63,
    "fightIQ": 69
  },
  "record": {
    "wins": 8,
    "losses": 3,
    "draws": 1
  },
  "age": 28,
  "seasonsActive": 3,
  "trainingHours": 50,
  "cash": 1250,
  "cosmetics": {
    "appearance": "default",
    "celebration": "default",
    "corner": "default"
  },
  "lastFight": "2024-10-01",
  "created": "2024-01-15"
}
```

### League Schema
```json
{
  "id": "league_uuid",
  "name": "Elite Combat League",
  "type": "seasonal",
  "settings": {
    "maxFighters": 10,
    "weeklySchedule": "sunday",
    "matchmakingType": "swiss",
    "agingEnabled": true,
    "seasonLength": 12
  },
  "currentWeek": 8,
  "season": 2,
  "fighters": ["fighter_uuid1", "fighter_uuid2"],
  "standings": [
    {
      "fighterId": "fighter_uuid1",
      "wins": 6,
      "losses": 1,
      "points": 18
    }
  ],
  "schedule": [
    {
      "week": 8,
      "fights": [
        {
          "fighter1": "fighter_uuid1",
          "fighter2": "fighter_uuid2",
          "scheduled": "2024-10-06"
        }
      ]
    }
  ]
}
```

### Fight Schema
```json
{
  "id": "fight_uuid",
  "leagueId": "league_uuid",
  "week": 8,
  "fighter1": {
    "id": "fighter_uuid1",
    "tactics": {
      "distance": 75,
      "tempo": 60,
      "tactics": 40,
      "grapple": 25
    }
  },
  "fighter2": {
    "id": "fighter_uuid2", 
    "tactics": {
      "distance": 30,
      "tempo": 80,
      "tactics": 70,
      "grapple": 60
    }
  },
  "result": {
    "winner": "fighter_uuid1",
    "method": "Decision",
    "round": 3,
    "time": "5:00"
  },
  "fightLog": [
    {
      "round": 1,
      "phase": 1,
      "action": "Fighter1 lands jab",
      "damage": 8,
      "stamina": {"f1": 92, "f2": 88}
    }
  ],
  "seed": "fight_seed_12345",
  "simulated": "2024-10-06T15:30:00Z"
}
```

### Training Log Schema
```json
{
  "fighterId": "fighter_uuid",
  "week": 8,
  "season": 2,
  "allocation": {
    "strength": 8,
    "speed": 12,
    "skill": 10,
    "endurance": 8,
    "chin": 4,
    "fightIQ": 8
  },
  "totalHours": 50,
  "bonusHours": 10,
  "efficiency": 0.95,
  "gains": {
    "strength": 1.2,
    "speed": 1.8,
    "skill": 1.5,
    "endurance": 1.2,
    "chin": 0.6,
    "fightIQ": 1.2
  },
  "penalties": {
    "nextFightStamina": -0.05
  },
  "submitted": "2024-10-01T12:00:00Z"
}
```

### Match Result Schema
```json
{
  "fightId": "fight_uuid",
  "summary": {
    "winner": "Fighter Name",
    "loser": "Opponent Name", 
    "method": "TKO",
    "round": 2,
    "time": "3:47",
    "totalTime": "8:47"
  },
  "statistics": {
    "fighter1": {
      "significantStrikes": 23,
      "takedowns": 1,
      "submissions": 0,
      "knockdowns": 0,
      "damageDealt": 67,
      "staminaUsed": 78
    },
    "fighter2": {
      "significantStrikes": 18,
      "takedowns": 0,
      "submissions": 0,
      "knockdowns": 1,
      "damageDealt": 45,
      "staminaUsed": 85
    }
  },
  "keyMoments": [
    {
      "round": 1,
      "time": "2:15",
      "event": "Fighter1 drops Fighter2 with overhand right"
    },
    {
      "round": 2,
      "time": "3:47", 
      "event": "Referee stops fight - TKO victory Fighter1"
    }
  ],
  "tendencies": {
    "fighter1": "Effective counter-punching, good takedown defense",
    "fighter2": "Struggled with pressure, low stamina in round 2"
  }
}
```

---

## 5. MVP Prototype (HTML/JS)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Proving Grounds MVP</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background: #1a1a1a; color: #fff; }
        .container { max-width: 1200px; margin: 0 auto; padding: 20px; }
        .header { text-align: center; margin-bottom: 30px; }
        .nav { display: flex; gap: 10px; justify-content: center; margin-bottom: 20px; }
        .nav button { padding: 10px 20px; background: #333; color: #fff; border: none; cursor: pointer; }
        .nav button.active { background: #007acc; }
        .section { display: none; }
        .section.active { display: block; }
        .fighter-card { background: #2a2a2a; padding: 20px; margin: 10px 0; border-radius: 8px; }
        .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin: 20px 0; }
        .stat-item { background: #333; padding: 15px; border-radius: 5px; text-align: center; }
        .slider-container { margin: 10px 0; }
        .slider { width: 100%; margin: 5px 0; }
        .training-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; }
        .fight-log { background: #2a2a2a; padding: 15px; margin: 10px 0; border-radius: 5px; max-height: 300px; overflow-y: auto; }
        .standings { background: #2a2a2a; padding: 20px; border-radius: 8px; }
        .standings table { width: 100%; border-collapse: collapse; }
        .standings th, .standings td { padding: 10px; text-align: left; border-bottom: 1px solid #444; }
        input, select, button { padding: 8px; margin: 5px; background: #333; color: #fff; border: 1px solid #555; }
        button { cursor: pointer; }
        button:hover { background: #555; }
        .progress-bar { background: #333; height: 20px; border-radius: 10px; overflow: hidden; margin: 5px 0; }
        .progress-fill { background: #007acc; height: 100%; transition: width 0.3s; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Proving Grounds</h1>
            <p>Fantasy Combat Sports Management</p>
        </div>

        <div class="nav">
            <button onclick="showSection('fighter')" class="active">Fighter</button>
            <button onclick="showSection('training')">Training</button>
            <button onclick="showSection('fight')">Fight</button>
            <button onclick="showSection('league')">League</button>
        </div>

        <!-- Fighter Section -->
        <div id="fighter" class="section active">
            <h2>Fighter Management</h2>
            <div class="fighter-card">
                <h3 id="fighterName">Create Your Fighter</h3>
                <div style="margin: 20px 0;">
                    <input type="text" id="nameInput" placeholder="Fighter Name" maxlength="20">
                    <select id="styleSelect">
                        <option value="Boxer">Boxer</option>
                        <option value="Wrestler">Wrestler</option>
                        <option value="BJJ">BJJ</option>
                        <option value="Kickboxer">Kickboxer</option>
                        <option value="Pressure">Pressure</option>
                        <option value="Counter">Counter</option>
                    </select>
                    <button onclick="createFighter()">Create Fighter</button>
                </div>
                
                <div class="stats-grid">
                    <div class="stat-item">
                        <h4>Strength</h4>
                        <div class="progress-bar">
                            <div id="strengthBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="strengthValue">50</span>
                    </div>
                    <div class="stat-item">
                        <h4>Speed</h4>
                        <div class="progress-bar">
                            <div id="speedBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="speedValue">50</span>
                    </div>
                    <div class="stat-item">
                        <h4>Skill</h4>
                        <div class="progress-bar">
                            <div id="skillBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="skillValue">50</span>
                    </div>
                    <div class="stat-item">
                        <h4>Endurance</h4>
                        <div class="progress-bar">
                            <div id="enduranceBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="enduranceValue">50</span>
                    </div>
                    <div class="stat-item">
                        <h4>Chin</h4>
                        <div class="progress-bar">
                            <div id="chinBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="chinValue">50</span>
                    </div>
                    <div class="stat-item">
                        <h4>Fight IQ</h4>
                        <div class="progress-bar">
                            <div id="fightIQBar" class="progress-fill" style="width: 50%"></div>
                        </div>
                        <span id="fightIQValue">50</span>
                    </div>
                </div>

                <div style="margin-top: 20px;">
                    <p><strong>Record:</strong> <span id="record">0-0-0</span></p>
                    <p><strong>Training Hours:</strong> <span id="trainingHours">50</span></p>
                    <p><strong>Cash:</strong> $<span id="cash">500</span></p>
                </div>
            </div>
        </div>

        <!-- Training Section -->
        <div id="training" class="section">
            <h2>Training Camp</h2>
            <div class="training-grid">
                <div>
                    <h3>Allocate Training Hours</h3>
                    <p>Available Hours: <span id="availableHours">50</span></p>
                    
                    <div class="slider-container">
                        <label>Strength: <span id="strengthHours">8</span> hours</label>
                        <input type="range" class="slider" id="strengthSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Speed: <span id="speedHours">8</span> hours</label>
                        <input type="range" class="slider" id="speedSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Skill: <span id="skillHours">8</span> hours</label>
                        <input type="range" class="slider" id="skillSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Endurance: <span id="enduranceHours">8</span> hours</label>
                        <input type="range" class="slider" id="enduranceSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Chin: <span id="chinHours">8</span> hours</label>
                        <input type="range" class="slider" id="chinSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Fight IQ: <span id="fightIQHours">8</span> hours</label>
                        <input type="range" class="slider" id="fightIQSlider" min="0" max="20" value="8" oninput="updateTraining()">
                    </div>
                    
                    <button onclick="submitTraining()">Submit Training</button>
                </div>
                
                <div>
                    <h3>Training Effects</h3>
                    <div id="trainingEffects">
                        <p>Balanced training - no penalties</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Fight Section -->
        <div id="fight" class="section">
            <h2>Fight Preparation</h2>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
                <div>
                    <h3>Fight Tactics</h3>
                    
                    <div class="slider-container">
                        <label>Distance: <span id="distanceValue">50</span> (Close ← → Long Range)</label>
                        <input type="range" class="slider" id="distanceSlider" min="0" max="100" value="50" oninput="updateTactics()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Tempo: <span id="tempoValue">50</span> (Patient ← → Aggressive)</label>
                        <input type="range" class="slider" id="tempoSlider" min="0" max="100" value="50" oninput="updateTactics()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Tactics: <span id="tacticsValue">50</span> (Technical ← → Brawling)</label>
                        <input type="range" class="slider" id="tacticsSlider" min="0" max="100" value="50" oninput="updateTactics()">
                    </div>
                    
                    <div class="slider-container">
                        <label>Grappling: <span id="grappleValue">50</span> (Striking ← → Wrestling)</label>
                        <input type="range" class="slider" id="grappleSlider" min="0" max="100" value="50" oninput="updateTactics()">
                    </div>
                    
                    <button onclick="simulateFight()">Simulate Fight</button>
                </div>
                
                <div>
                    <h3>Opponent Scouting</h3>
                    <div id="opponentInfo">
                        <p><strong>Opponent:</strong> AI Fighter</p>
                        <p><strong>Style:</strong> Boxer</p>
                        <p><strong>Record:</strong> 5-3-0</p>
                        <p><strong>Tendencies:</strong> Aggressive striker, weak ground game</p>
                    </div>
                </div>
            </div>
            
            <div class="fight-log" id="fightLog">
                <h3>Fight Results</h3>
                <p>Set your tactics and simulate a fight to see results...</p>
            </div>
        </div>

        <!-- League Section -->
        <div id="league" class="section">
            <h2>League Standings</h2>
            <div class="standings">
                <table>
                    <thead>
                        <tr>
                            <th>Rank</th>
                            <th>Fighter</th>
                            <th>Record</th>
                            <th>Points</th>
                        </tr>
                    </thead>
                    <tbody id="standingsTable">
                        <tr>
                            <td>1</td>
                            <td>Your Fighter</td>
                            <td>0-0-0</td>
                            <td>0</td>
                        </tr>
                        <tr>
                            <td>2</td>
                            <td>AI Fighter 1</td>
                            <td>0-0-0</td>
                            <td>0</td>
                        </tr>
                        <tr>
                            <td>3</td>
                            <td>AI Fighter 2</td>
                            <td>0-0-0</td>
                            <td>0</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        // Game State
        let gameState = {
            fighter: {
                name: "New Fighter",
                style: "Boxer",
                stats: {
                    strength: 50,
                    speed: 50,
                    skill: 50,
                    endurance: 50,
                    chin: 50,
                    fightIQ: 50
                },
                record: { wins: 0, losses: 0, draws: 0 },
                trainingHours: 50,
                cash: 500
            },
            training: {
                allocation: {
                    strength: 8,
                    speed: 8,
                    skill: 8,
                    endurance: 8,
                    chin: 8,
                    fightIQ: 8
                }
            },
            tactics: {
                distance: 50,
                tempo: 50,
                tactics: 50,
                grapple: 50
            }
        };

        // Load saved game state
        function loadGame() {
            const saved = localStorage.getItem('provingGrounds');
            if (saved) {
                gameState = JSON.parse(saved);
                updateUI();
            }
        }

        // Save game state
        function saveGame() {
            localStorage.setItem('provingGrounds', JSON.stringify(gameState));
        }

        // Show section
        function showSection(sectionName) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.querySelectorAll('.nav button').forEach(b => b.classList.remove('active'));
            document.getElementById(sectionName).classList.add('active');
            event.target.classList.add('active');
        }

        // Create fighter
        function createFighter() {
            const name = document.getElementById('nameInput').value || 'New Fighter';
            const style = document.getElementById('styleSelect').value;
            
            gameState.fighter.name = name;
            gameState.fighter.style = style;
            
            // Apply style bonuses
            applyStyleBonuses(style);
            updateUI();
            saveGame();
        }

        // Apply style bonuses
        function applyStyleBonuses(style) {
            const bonuses = {
                'Boxer': { speed: 10, skill: 5 },
                'Wrestler': { strength: 10, endurance: 5 },
                'BJJ': { skill: 15 },
                'Kickboxer': { speed: 5, skill: 5 },
                'Pressure': { endurance: 10, chin: 5 },
                'Counter': { fightIQ: 15 }
            };
            
            // Reset to base 50
            Object.keys(gameState.fighter.stats).forEach(stat => {
                gameState.fighter.stats[stat] = 50;
            });
            
            // Apply bonuses
            if (bonuses[style]) {
                Object.entries(bonuses[style]).forEach(([stat, bonus]) => {
                    gameState.fighter.stats[stat] += bonus;
                });
            }
        }

        // Update training allocation
        function updateTraining() {
            const stats = ['strength', 'speed', 'skill', 'endurance', 'chin', 'fightIQ'];
            let totalHours = 0;
            
            stats.forEach(stat => {
                const value = parseInt(document.getElementById(stat + 'Slider').value);
                gameState.training.allocation[stat] = value;
                document.getElementById(stat + 'Hours').textContent = value;
                totalHours += value;
            });
            
            document.getElementById('availableHours').textContent = 50 - totalHours;
            
            // Update training effects
            updateTrainingEffects();
        }

        // Update training effects
        function updateTrainingEffects() {
            const effects = [];
            const allocation = gameState.training.allocation;
            
            if (allocation.strength > 15) {
                effects.push("Heavy strength training: -10% stamina efficiency next fight");
            }
            if (allocation.speed > 15) {
                effects.push("Speed focus: -5% power next fight");
            }
            if (effects.length === 0) {
                effects.push("Balanced training - no penalties");
            }
            
            document.getElementById('trainingEffects').innerHTML = effects.map(e => `<p>${e}</p>`).join('');
        }

        // Submit training
        function submitTraining() {
            const allocation = gameState.training.allocation;
            const totalHours = Object.values(allocation).reduce((sum, hours) => sum + hours, 0);
            
            if (totalHours > 50) {
                alert('Cannot exceed 50 training hours!');
                return;
            }
            
            // Apply training gains
            Object.entries(allocation).forEach(([stat, hours]) => {
                const currentStat = gameState.fighter.stats[stat];
                const efficiency = calculateTrainingEfficiency(currentStat);
                const gain = hours * efficiency;
                gameState.fighter.stats[stat] = Math.min(100, currentStat + gain);
            });
            
            updateUI();
            saveGame();
            alert('Training completed! Stats updated.');
        }

        // Calculate training efficiency
        function calculateTrainingEfficiency(currentStat) {
            if (currentStat < 60) return 0.5;
            if (currentStat < 80) return 0.3;
            return 0.1;
        }

        // Update fight tactics
        function updateTactics() {
            const tactics = ['distance', 'tempo', 'tactics', 'grapple'];
            
            tactics.forEach(tactic => {
                const value = parseInt(document.getElementById(tactic + 'Slider').value);
                gameState.tactics[tactic] = value;
                document.getElementById(tactic + 'Value').textContent = value;
            });
        }

        // Simulate fight
        function simulateFight() {
            const fighter = gameState.fighter;
            const opponent = generateOpponent();
            const result = runFightSimulation(fighter, opponent, gameState.tactics);
            
            displayFightResult(result);
            updateRecord(result.winner === 'player');
            saveGame();
        }

        // Generate AI opponent
        function generateOpponent() {
            const styles = ['Boxer', 'Wrestler', 'BJJ', 'Kickboxer', 'Pressure', 'Counter'];
            const style = styles[Math.floor(Math.random() * styles.length)];
            
            return {
                name: 'AI Fighter',
                style: style,
                stats: {
                    strength: 45 + Math.random() * 20,
                    speed: 45 + Math.random() * 20,
                    skill: 45 + Math.random() * 20,
                    endurance: 45 + Math.random() * 20,
                    chin: 45 + Math.random() * 20,
                    fightIQ: 45 + Math.random() * 20
                }
            };
        }

        // Run fight simulation
        function runFightSimulation(fighter, opponent, tactics) {
            const log = [];
            let fighterHP = 100;
            let opponentHP = 100;
            let fighterStamina = 100;
            let opponentStamina = 100;
            
            // Simulate 3 rounds
            for (let round = 1; round <= 3; round++) {
                log.push(`\n=== ROUND ${round} ===`);
                
                // 5 phases per round
                for (let phase = 1; phase <= 5; phase++) {
                    // Fighter action
                    const fighterAction = selectAction(fighter, tactics);
                    const fighterDamage = calculateDamage(fighter, opponent, fighterAction);
                    opponentHP -= fighterDamage;
                    fighterStamina -= 8;
                    
                    if (fighterDamage > 0) {
                        log.push(`${fighter.name} ${fighterAction} for ${fighterDamage.toFixed(1)} damage`);
                    }
                    
                    if (opponentHP <= 0) {
                        log.push(`${opponent.name} is knocked out!`);
                        return { winner: 'player', method: 'KO', round, log };
                    }
                    
                    // Opponent action
                    const opponentTactics = { distance: 50, tempo: 60, tactics: 40, grapple: 30 };
                    const opponentAction = selectAction(opponent, opponentTactics);
                    const opponentDamage = calculateDamage(opponent, fighter, opponentAction);
                    fighterHP -= opponentDamage;
                    opponentStamina -= 8;
                    
                    if (opponentDamage > 0) {
                        log.push(`${opponent.name} ${opponentAction} for ${opponentDamage.toFixed(1)} damage`);
                    }
                    
                    if (fighterHP <= 0) {
                        log.push(`${fighter.name} is knocked out!`);
                        return { winner: 'opponent', method: 'KO', round, log };
                    }
                }
                
                // End of round
                fighterStamina = Math.min(100, fighterStamina + 20);
                opponentStamina = Math.min(100, opponentStamina + 20);
                log.push(`End of round ${round} - ${fighter.name}: ${fighterHP.toFixed(1)} HP, ${opponent.name}: ${opponentHP.toFixed(1)} HP`);
            }
            
            // Decision
            const winner = fighterHP > opponentHP ? 'player' : 'opponent';
            log.push(`\nFight goes to decision - Winner: ${winner === 'player' ? fighter.name : opponent.name}`);
            
            return { winner, method: 'Decision', round: 3, log };
        }

        // Select action based on tactics
        function selectAction(fighter, tactics) {
            const actions = ['strikes', 'grapples', 'defends'];
            const weights = [
                (100 - tactics.grapple) * 0.01,
                tactics.grapple * 0.01,
                (100 - tactics.tempo) * 0.01
            ];
            
            const random = Math.random();
            let cumulative = 0;
            
            for (let i = 0; i < actions.length; i++) {
                cumulative += weights[i];
                if (random < cumulative) {
                    return actions[i];
                }
            }
            
            return actions[0];
        }

        // Calculate damage
        function calculateDamage(attacker, defender, action) {
            if (action === 'defends') return 0;
            
            const baseDamage = attacker.stats.strength * 0.3 + attacker.stats.skill * 0.2;
            const accuracy = (attacker.stats.speed - defender.stats.speed + 50) / 100;
            const defense = defender.stats.chin * 0.4;
            
            let damage = baseDamage * accuracy - defense;
            damage *= (0.85 + Math.random() * 0.3); // ±15% variance
            
            return Math.max(0, damage);
        }

        // Display fight result
        function displayFightResult(result) {
            const logElement = document.getElementById('fightLog');
            logElement.innerHTML = `<h3>Fight Results</h3><pre>${result.log.join('\n')}</pre>`;
        }

        // Update fighter record
        function updateRecord(won) {
            if (won) {
                gameState.fighter.record.wins++;
                gameState.fighter.cash += 150;
            } else {
                gameState.fighter.record.losses++;
                gameState.fighter.cash += 100;
            }
            updateUI();
        }

        // Update UI
        function updateUI() {
            const fighter = gameState.fighter;
            
            // Fighter info
            document.getElementById('fighterName').textContent = fighter.name;
            document.getElementById('record').textContent = `${fighter.record.wins}-${fighter.record.losses}-${fighter.record.draws}`;
            document.getElementById('trainingHours').textContent = fighter.trainingHours;
            document.getElementById('cash').textContent = fighter.cash;
            
            // Stats
            Object.entries(fighter.stats).forEach(([stat, value]) => {
                document.getElementById(stat + 'Value').textContent = Math.round(value);
                document.getElementById(stat + 'Bar').style.width = value + '%';
            });
        }

        // Initialize game
        loadGame();
        updateUI();
    </script>
</body>
</html>
```

---

## 6. Balance Tables

### Styles Matrix (Advantage Percentages)
| Attacker ↓ | Boxer | Wrestler | BJJ | Kickboxer | Pressure | Counter |
|------------|-------|----------|-----|-----------|----------|---------|
| **Boxer** | 0% | -15% | +5% | +10% | -5% | +5% |
| **Wrestler** | +15% | 0% | -10% | +5% | +10% | -5% |
| **BJJ** | -5% | +10% | 0% | -15% | +5% | +10% |
| **Kickboxer** | -10% | -5% | +15% | 0% | -10% | +5% |
| **Pressure** | +5% | -10% | -5% | +10% | 0% | -15% |
| **Counter** | -5% | +5% | -10% | -5% | +15% | 0% |

### Training Hours Efficiency Curve
```
Stat Range | Hours per Point | Cumulative Cost
0-60       | 2.0            | 120H total
60-70      | 3.0            | 150H total  
70-80      | 5.0            | 200H total
80-90      | 8.0            | 280H total
90-100     | 12.0           | 400H total
```

### XP/Aging Table (Per Season)
| Age Range | Speed Loss | Endurance Loss | Skill Gain | Fight IQ Gain |
|-----------|------------|----------------|------------|---------------|
| 18-25     | 0          | 0              | +2         | +1            |
| 26-30     | -1         | 0              | +1         | +2            |
| 31-35     | -1         | -1             | 0          | +1            |
| 36-40     | -2         | -1             | -1         | 0             |
| 41+       | -2         | -2             | -1         | -1            |

### Legacy Carryover Rules
**Between Leagues**:
- All stats normalized to league median ±10
- Record resets to 0-0-0
- Cash carries over fully
- Cosmetics carry over
- Age continues naturally

**Season Transitions**:
- Stats carry over with aging effects
- Training hours reset to 50
- Cash bonus based on final ranking
- Record archived, new season starts 0-0-0

---

## 7. Post-Fight Report Template

```
=== FIGHT REPORT ===
Winner: [Fighter Name] defeats [Opponent Name]
Method: [KO/TKO/Submission/Decision]
Time: Round [X] at [X:XX] ([Total Time])

FIGHT SUMMARY:
[2-3 sentence narrative of key moments and turning points]

STATISTICS:
                    Winner    Loser
Significant Strikes   23       18
Takedowns             1        0  
Submissions           0        0
Knockdowns            0        1
Total Damage         67       45
Stamina Used         78%      85%

KEY MOMENTS:
• Round 1, 2:15 - [Winner] drops [Loser] with overhand right
• Round 2, 3:47 - Referee stops fight, TKO victory [Winner]

PERFORMANCE ANALYSIS:
Winner: [Effective counter-punching, good takedown defense, maintained pace]
Loser: [Struggled with pressure, low stamina in round 2, predictable patterns]

TACTICAL INSIGHTS:
• [Winner]'s distance control neutralized [Loser]'s grappling
• Tempo advantage became decisive in later rounds
• Style matchup favored [Winner] by approximately 10%

EARNINGS:
Winner: +150 Cash, +3 League Points
Loser: +100 Cash, +0 League Points

NEXT STEPS:
• Review opponent tendencies for future matchups
• Consider tactical adjustments based on performance
• Plan training focus for upcoming fights
```

---

## 8. Visual Wireframe (Text)

### Main Menu Layout
```
[HEADER: Proving Grounds Logo + Tagline]
[NAV BAR: Fighter | Training | Fight | League | Settings]

FIGHTER TAB:
┌─────────────────────────────────────────┐
│ Fighter Card                            │
│ ┌─────────┐ Name: [Input Field]         │
│ │ Avatar  │ Style: [Dropdown]           │
│ │ Image   │ Record: X-X-X               │
│ └─────────┘ Cash: $XXX                  │
│                                         │
│ Stats Grid (3x2):                       │
│ [STR: ████████░░] [SPD: ██████░░░░]     │
│ [SKL: ███████░░░] [END: █████████░]     │
│ [CHN: ██████░░░░] [IQ:  ████████░░]     │
└─────────────────────────────────────────┘
```

### Training Screen Layout
```
TRAINING TAB:
┌─────────────────┬─────────────────────┐
│ Hour Allocation │ Training Effects    │
│                 │                     │
│ Available: 50H  │ Current Plan:       │
│                 │ • Balanced training │
│ STR: [====] 8H  │ • No penalties      │
│ SPD: [====] 8H  │                     │
│ SKL: [====] 8H  │ Projected Gains:    │
│ END: [====] 8H  │ • STR: +1.2         │
│ CHN: [====] 8H  │ • SPD: +1.2         │
│ IQ:  [====] 8H  │ • etc...            │
│                 │                     │
│ [Submit Training] │                   │
└─────────────────┴─────────────────────┘
```

### Fight Simulation Layout
```
FIGHT TAB:
┌─────────────────┬─────────────────────┐
│ Tactics Setup   │ Opponent Scouting   │
│                 │                     │
│ Distance:       │ Name: AI Fighter    │
│ [Close ████ Long] │ Style: Boxer        │
│                 │ Record: 5-3-0       │
│ Tempo:          │ Tendencies:         │
│ [Patient ██ Agg] │ • Aggressive       │
│                 │ • Weak ground game  │
│ Tactics:        │                     │
│ [Tech ████ Brawl] │                   │
│                 │                     │
│ Grappling:      │                     │
│ [Strike ██ Grap] │                    │
│                 │                     │
│ [Simulate Fight] │                    │
└─────────────────┴─────────────────────┘

┌─────────────────────────────────────────┐
│ Fight Log (Scrollable)                  │
│ === ROUND 1 ===                        │
│ Fighter lands jab for 8.2 damage       │
│ Opponent grapples for 12.1 damage      │
│ Fighter defends successfully            │
│ ...                                     │
└─────────────────────────────────────────┘
```

### League Standings Layout
```
LEAGUE TAB:
┌─────────────────────────────────────────┐
│ League: Elite Combat League             │
│ Week: 8/12 | Season: 2                  │
│                                         │
│ Standings Table:                        │
│ Rank | Fighter    | Record | Points     │
│ ─────┼────────────┼────────┼──────      │
│  1   │ Your Name  │  6-1-0 │   18       │
│  2   │ AI Fighter │  5-2-0 │   15       │
│  3   │ Bot Alpha  │  4-3-0 │   12       │
│  4   │ etc...     │  etc   │   etc      │
│                                         │
│ Next Fight: vs AI Fighter (Sunday)      │
│ [View Schedule] [League Settings]       │
└─────────────────────────────────────────┘
```

---

## 9. Expansion Roadmap

### Phase 1: Core Enhancement (Months 1-3)
- **Advanced AI**: Opponent learning, adaptive tactics
- **Style Refinement**: Muay Thai, Karate, Hybrid stances
- **Training Specialization**: Gym partnerships, coach bonuses
- **Enhanced UI**: Animated fight visualization, stat comparisons

### Phase 2: Social Features (Months 4-6)
- **Multi-Fighter Management**: Stable of 3-5 fighters
- **Gym System**: Shared training facilities, team bonuses
- **Live Spectating**: Watch friends' fights in real-time
- **Tournament Mode**: Single-elimination brackets

### Phase 3: Competitive Expansion (Months 7-9)
- **Global Leaderboards**: Cross-league rankings
- **Championship Series**: Seasonal tournaments
- **Advanced Analytics**: Performance tracking, tendency analysis
- **Mobile Optimization**: Responsive design, touch controls

### Phase 4: Content & Depth (Months 10-12)
- **Career Mode**: Multi-season progression, retirement
- **Historical Records**: Hall of Fame, legacy tracking
- **Advanced Customization**: Detailed fighter appearance
- **Mod Support**: Community-created content

### Long-term Vision (Year 2+)
- **Real-time Multiplayer**: Simultaneous league management
- **VR Integration**: Immersive fight viewing
- **Machine Learning**: Personalized opponent generation
- **Esports Integration**: Competitive league structure

### Technical Milestones
- **Database Migration**: Move from localStorage to cloud saves
- **API Development**: RESTful backend for multiplayer
- **Performance Optimization**: Sub-100ms fight simulation
- **Cross-platform**: Web, mobile, desktop clients

### Monetization Strategy (Post-Launch)
- **Premium Cosmetics**: Advanced customization options
- **League Hosting**: Private league management tools
- **Analytics Plus**: Advanced statistics and insights
- **No Pay-to-Win**: All gameplay advantages remain earned

---

## Implementation Notes

This design document provides a complete foundation for building Proving Grounds as a fantasy combat sports management game. The MVP prototype demonstrates core mechanics while the expansion roadmap ensures long-term growth potential.

Key design principles maintained throughout:
- **No pay-to-win mechanics** - all advantages earned through play
- **Balanced complexity** - deep enough for strategy, simple enough for accessibility  
- **Deterministic simulation** - consistent results with controlled variance
- **Social engagement** - league-based competition drives retention
- **Progressive depth** - systems scale from casual to hardcore engagement

The modular architecture allows for incremental development while maintaining system coherence. Each component can be developed and tested independently before integration.