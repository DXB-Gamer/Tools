<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Pocket Money Tracker</title>
</head>
<body>

  <!-- # Site Icon in Top Left -->
  <img src="https://github.com/user-attachments/assets/8dd03411-8873-45d3-8d03-79c7f754e5b3" 
       alt="PMT Icon" 
       style="position: fixed; top: 10px; left: 10px; width: 50px; height: 50px; z-index: 1000;">

  <!-- # Your Pocket Money Tracker code goes here -->
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pocket Money Pro</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
:root {
    --bg-color: #0f0c1a;
    --border-color: #00bfff;
    --hover-border: #00ff7f;
}
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: var(--bg-color);
    color: white;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30px;
}
.container {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}
.card, .calculator, .owsl, .settings {
    background: #1e1a2b;
    padding: 24px;
    border-radius: 16px;
    border: 2px solid var(--border-color);
    box-shadow: 0 0 8px rgba(0,191,255,0.3);
    transition: border-color 0.3s, box-shadow 0.3s;
}
.card:hover,
.calculator:hover,
.owsl:hover,
.settings:hover {
    border-color: var(--hover-border);
    box-shadow: 0 0 18px var(--hover-border), 0 0 30px var(--hover-border);
}
.card { width: 430px; }
.calculator { width: 260px; background: #1b2a44; }
input, select {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    border-radius: 8px;
    border: none;
    background: #2a2540;
    color: white;
}
button {
    width: 100%;
    margin-top: 14px;
    padding: 12px;
    border-radius: 10px;
    border: none;
    background: #e85b78;
    color: white;
    font-weight: bold;
    cursor: pointer;
}
canvas {
    margin-top: 20px;
    background: #241f38;
    border-radius: 12px;
}
.calc-display {
    background: #0d1b2a;
    padding: 10px;
    border-radius: 8px;
    text-align: right;
    margin-bottom: 10px;
}
.calc-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 6px;
}
.calc-btn {
    padding: 10px;
    background: #3b82f6;
    border: none;
    border-radius: 6px;
    color: white;
}
.owsl, .settings {
    margin-top: 25px;
    width: 720px;
}
.settings-row {
    display: flex;
    gap: 15px;
    align-items: center;
}
</style>
</head>
<body>

<div class="container">

<div class="card">
    <h2>Pocket Money</h2>
    <input type="number" id="monthlyMoney" value="50">
    <select id="currency">
        <option>AED</option><option>USD</option><option>CAD</option>
        <option>EUR</option><option>GBP</option><option>PKR</option>
        <option>INR</option><option>JPY</option><option>SAR</option>
        <option>QAR</option><option>AUD</option>
    </select>
    <input type="text" id="convertTo" placeholder="Convert to any currency">
    <button onclick="calculate()">Calculate</button>
    <canvas id="moneyChart" height="200"></canvas>
</div>

<div class="calculator">
    <div class="calc-display" id="calcDisplay">0</div>
    <div class="calc-grid">
        <button class="calc-btn" onclick="press('7')">7</button>
        <button class="calc-btn" onclick="press('8')">8</button>
        <button class="calc-btn" onclick="press('9')">9</button>
        <button class="calc-btn" onclick="press('/')">÷</button>
        <button class="calc-btn" onclick="press('4')">4</button>
        <button class="calc-btn" onclick="press('5')">5</button>
        <button class="calc-btn" onclick="press('6')">6</button>
        <button class="calc-btn" onclick="press('*')">×</button>
        <button class="calc-btn" onclick="press('1')">1</button>
        <button class="calc-btn" onclick="press('2')">2</button>
        <button class="calc-btn" onclick="press('3')">3</button>
        <button class="calc-btn" onclick="press('-')">−</button>
        <button class="calc-btn" onclick="press('0')">0</button>
        <button class="calc-btn" onclick="clearCalc()">C</button>
        <button class="calc-btn" onclick="equals()">=</button>
        <button class="calc-btn" onclick="press('+')">+</button>
    </div>
</div>

</div>

<div class="owsl">
    <h3>OWSL(C) Translator</h3>
    <input type="text" id="owslInput" placeholder="Enter English text">
    <button onclick="translateOWSL()">Translate</button>
    <p id="owslOutput"></p>
</div>

<div class="settings">
    <h3>Appearance Settings</h3>
    <div class="settings-row">
        <label>Background</label>
        <input type="color" onchange="setBG(this.value)">
        <label>Border</label>
        <input type="color" onchange="setBorder(this.value)">
        <label>Hover</label>
        <input type="color" onchange="setHover(this.value)">
    </div>
</div>

<script>
let chart, display = "";

async function calculate() {
    const m = Number(monthlyMoney.value);
    const base = currency.value;
    const to = convertTo.value.toUpperCase();
    let values = [m*6, m*12, m*60, m*120];
    let label = base;
    if (to) {
        try {
            const r = await fetch(`https://api.exchangerate-api.com/v4/latest/${base}`);
            const d = await r.json();
            if (d.rates[to]) {
                values = values.map(v => (v * d.rates[to]).toFixed(2));
                label = to;
            }
        } catch {}
    }
    if (chart) chart.destroy();
    chart = new Chart(moneyChart, {
        type: "line",
        data: {
            labels: ["6 months", "1 year", "5 years", "10 years"],
            datasets: [{
                label: `Savings (${label})`,
                data: values,
                borderColor: "#00bfff",
                backgroundColor: "rgba(0,191,255,0.2)",
                fill: true,
                tension: 0.4
            }]
        }
    });
}

function press(v){ display += v; calcDisplay.innerText = display; }
function clearCalc(){ display = ""; calcDisplay.innerText = "0"; }
function equals(){ try{ display = eval(display); }catch{ display="Error"; } calcDisplay.innerText = display; }

function translateOWSL(){
    owslOutput.innerText = owslInput.value
        .split(" ")
        .map(w => w.split("").reverse().join(""))
        .join(" ");
}

function setBG(c){ document.documentElement.style.setProperty('--bg-color', c); }
function setBorder(c){ document.documentElement.style.setProperty('--border-color', c); }
function setHover(c){ document.documentElement.style.setProperty('--hover-border', c); }

calculate();
</script>

</body>
</html>
  <!-- # Our Team Section -->
  <h2 style="color: #1e90ff; text-align: center; margin-top: 50px;">Our Team</h2>

  <!-- # Developers Button -->
  <a href="https://github.com/DXB-Gamer" target="_blank" class="profile-btn">Developers Profile</a>

  <!-- # Designers/Artists Button (placeholder for later) -->
  <!-- <a href="https://github.com/DesignerName" target="_blank" class="profile-btn">Designers Profile</a> -->

  <style>
    .profile-btn {
      display: inline-block;
      padding: 12px 25px;
      font-weight: bold;
      color: #fff;
      text-decoration: none;
      border: 2px solid #1e90ff;
      border-radius: 8px;
      background: linear-gradient(90deg, rgba(30,144,255,0.2), rgba(30,144,255,0.1));
      box-shadow: 0 0 10px #1e90ff, 0 0 20px #00ffff;
      transition: all 0.3s ease;
      margin: 10px; /* space between buttons */
    }
    .profile-btn:hover {
      color: #00ffff;
      box-shadow: 0 0 20px #1e90ff, 0 0 40px #00ffff, 0 0 60px #00ffff;
      transform: scale(1.1);
    }
  </style>

</body>
</html>
