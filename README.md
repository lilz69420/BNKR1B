# BNKR1B
A real time cash to banker converter. V1.1
<!DOCTYPE html>
<html>
  <head>
    <title>Hello, World!</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
      <h1 class="title">Hello World! </h1>
      <p id="currentTime"></p>
      <script src="script.js"></script>
  </body>
</html><!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>BNKR 1B live (Made by @Lilz69420) — beta.V1.1</title>

  <!-- Dark tab styling + custom BNKR favicon -->
  <meta name="theme-color" content="#020603" />
  <meta name="color-scheme" content="dark" />
  <link rel="icon" type="image/svg+xml" href='data:image/svg+xml,
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
    <rect width="100" height="100" fill="black"/>
    <text x="50" y="65" font-size="40" font-family="monospace"
          fill="%2300ff88" text-anchor="middle">BNKR</text>
  </svg>' />

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    :root {
      --bg: #020603;
      --phosphor: #00ff88;
      --white: #ffffff;
      --border: #0c2b1b;
      --glass: rgba(0, 255, 136, 0.06);
    }

    /* Page layout */
    body {
      background: var(--bg);
      color: white;
      font-family: "VT323", monospace;
      display: grid;
      place-items: center;
      min-height: 100vh;
      margin: 0;
      padding: 20px;
      gap: 8px;
    }

    /* Top banner title */
    .title {
      color: var(--white);
      text-shadow: 0 0 6px #fff;
      margin: 0 0 6px 0;
      text-align: center;
      font-size: 1.6rem;
    }

    /* Small clock under banner */
    #currentTime {
      color: var(--phosphor);
      margin: 0 0 10px 0;
      font-size: 1.05rem;
      opacity: 0.95;
    }

    /* Main screen */
    .screen {
      position: relative;
      padding: 20px 20px 80px;
      border: 2px solid var(--border);
      border-radius: 12px;
      background: #050c06;
      width: min(900px, 95vw);
      box-shadow: 0 0 12px rgba(0,255,136,0.6);
    }

    h1 {
      font-size: 2rem;
      margin-bottom: 1rem;
      color: var(--white);
      text-shadow: 0 0 6px #fff;
    }

    .line { margin: 8px 0; font-size: 1.2rem; }

    .ratio {
      font-size: 1.8rem;
      color: var(--white);
      text-shadow: 0 0 4px #fff;
    }

    .small {
      font-size: 0.9rem;
      color: var(--phosphor);
      margin-top: 10px;
    }

    canvas {
      margin-top: 20px;
      background: #000;
      border: 1px solid var(--border);
      border-radius: 8px;
    }

    /* Cash box (top-right) */
    .cash-box {
      position: absolute;
      top: 16px; right: 16px;
      width: 300px; padding: 12px;
      border: 1px solid var(--border);
      border-radius: 10px;
      background: var(--glass);
      backdrop-filter: blur(3px);
      box-shadow: 0 0 8px rgba(0,255,136,0.25);
    }
    .cash-box h2 {
      margin: 0 0 8px 0;
      font-size: 1.2rem;
      color: var(--white);
      text-shadow: 0 0 4px #fff;
    }
    .row {
      display: flex;
      gap: 8px;
      margin: 8px 0;
    }
    .row input {
      flex: 1;
      background: #000;
      color: var(--white);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 8px 10px;
      font-family: "VT323", monospace;
      font-size: 1rem;
      outline: none;
      box-shadow: 0 0 6px rgba(0,255,136,0.15) inset;
    }
    .row button {
      background: #000;
      color: var(--phosphor);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 8px 12px;
      font-family: "VT323", monospace;
      font-size: 1rem;
      cursor: pointer;
    }
    .row button:hover { box-shadow: 0 0 8px rgba(0,255,136,0.35); }
    .cash-results { font-size: 1rem; line-height: 1.25rem; }
    .cash-results .muted { opacity: 0.9; }
    .cash-results .bold { color: var(--white); text-shadow: 0 0 3px #fff; }

    @media (max-width: 640px) {
      .cash-box { position: static; width: 100%; margin-bottom: 12px; }
      .screen { padding-top: 10px; }
    }
  </style>
</head>
<body>
  <!-- Top banner -->
  <h1 class="title">BNKR 1B live (Made by @Lilz69420) — beta.V1.1</h1>
  <p id="currentTime"></p>

  <!-- Terminal screen -->
  <div class="screen">
    <!-- Cash → 1B MC -->
    <div class="cash-box">
      <h2>Cash → 1B MC</h2>
      <div class="row">
        <input id="cashInput" type="number" min="0" step="0.01" placeholder="Enter USD (e.g., 2500)" />
        <button id="cashBtn">Calc</button>
      </div>
      <div class="cash-results" id="cashResults">
        <div class="muted">Enter an amount and press Enter.</div>
      </div>
    </div>

    <h1>$BNKR — 1B MC Monitor</h1>
    <div class="line" id="priceLine">Fetching...</div>
    <div class="line" id="ratioLine"></div>
    <div class="line ratio" id="ratioMultiple"></div>
    <div class="small" id="updatedAt"></div>
    <canvas id="bnkrChart" height="200"></canvas>
  </div>

  <script>
    // Clock under banner
    (function clock(){
      const el = document.getElementById("currentTime");
      const fmt = () => new Date().toLocaleString();
      const tick = () => el.textContent = fmt();
      tick();
      setInterval(tick, 1000);
    })();

    const priceLine = document.getElementById("priceLine");
    const ratioLine = document.getElementById("ratioLine");
    const ratioMultiple = document.getElementById("ratioMultiple");
    const updatedAt = document.getElementById("updatedAt");
    const cashInput = document.getElementById("cashInput");
    const cashBtn = document.getElementById("cashBtn");
    const cashResults = document.getElementById("cashResults");

    let lastPrice = null, lastImplied = null, lastMult = null;
    const targetCap = 1_000_000_000;
    const TICK_MS = 15000;   // 15s updates
    const MAX_POINTS = 480;  // ~2 hours of 15s points

    const fmtUSD = (n, d = 2) =>
      isFinite(n) ? `$${Number(n).toLocaleString(undefined,{minimumFractionDigits:d,maximumFractionDigits:d})}` : "—";

    // Chart setup
    const ctx = document.getElementById("bnkrChart").getContext("2d");
    const chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: [],
        datasets: [{
          label: "$BNKR Price (USD)",
          data: [],
          borderColor: "#00ff88",
          backgroundColor: "rgba(0,255,136,0.2)",
          pointRadius: 0,
          borderWidth: 2,
          tension: 0.2
        }]
      },
      options: {
        animation: { duration: 300 },
        plugins: { legend: { labels: { color: "#00ff88", font: { family: "VT323" } } } },
        scales: {
          x: { ticks: { color: "#00ff88", font: { family: "VT323" } } },
          y: { ticks: { color: "#00ff88", font: { family: "VT323" } } }
        }
      }
    });

    function appendPoint(time, price) {
      const label = time.toLocaleTimeString();
      const labels = chart.data.labels;
      if (labels.length && labels[labels.length - 1] === label) {
        chart.data.datasets[0].data[chart.data.datasets[0].data.length - 1] = price;
      } else {
        labels.push(label);
        chart.data.datasets[0].data.push(price);
        if (labels.length > MAX_POINTS) {
          labels.shift();
          chart.data.datasets[0].data.shift();
        }
      }
      chart.update();
    }

    function runCashCalc() {
      const cash = parseFloat(cashInput.value);
      if (!lastPrice || !lastImplied || !lastMult || !isFinite(cash) || cash <= 0) {
        cashResults.innerHTML = `<div class="muted">Enter a positive USD amount (data must be loaded).</div>`;
        return;
      }
      const tokensNow = cash / lastPrice;
      const futureValueAt1B = cash * lastMult; // same as tokensNow * lastImplied
      cashResults.innerHTML = `
        <div>Cash now: <span class="bold">${fmtUSD(cash)}</span></div>
        <div>Est. BNKR now: <span class="bold">${tokensNow.toLocaleString(undefined,{maximumFractionDigits:4})}</span> tokens</div>
        <div>Value if MC = $1B: <span class="bold">${fmtUSD(futureValueAt1B)}</span></div>
        <div>Gain multiple → <span class="bold">${lastMult.toFixed(2)}×</span></div>
      `;
    }

    cashBtn.addEventListener("click", runCashCalc);
    cashInput.addEventListener("keydown", (e) => { if (e.key === "Enter") runCashCalc(); });

    async function seedChart() {
      try {
        // Seed with ~last hour
        const chartUrl = "https://api.coingecko.com/api/v3/coins/bankercoin-2/market_chart?vs_currency=usd&days=1";
        const r = await fetch(chartUrl);
        const j = await r.json();
        const pts = j?.prices || [];
        const last60 = pts.slice(-60);
        chart.data.labels = last60.map(p => new Date(p[0]).toLocaleTimeString());
        chart.data.datasets[0].data = last60.map(p => Number(p[1]));
        chart.update();
      } catch (_) {/* silent */}
    }

    async function tick() {
      try {
        const url = "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&ids=bankercoin-2";
        const resp = await fetch(url, { cache: "no-store" });
        const [data] = await resp.json();

        const price = Number(data.current_price);
        const circSupply = Number(data.circulating_supply);
        const impliedPrice = targetCap / circSupply;
        const mult = impliedPrice / price;

        lastPrice = price;
        lastImplied = impliedPrice;
        lastMult = mult;

        priceLine.textContent = `Current price: ${fmtUSD(price, 6)} | Circ. Supply: ${circSupply.toLocaleString()}`;
        ratioLine.textContent = `Price if MC = $1,000,000,000: ${fmtUSD(impliedPrice, 6)}`;
        ratioMultiple.textContent = `${mult.toFixed(2)}× the current price`;
        updatedAt.textContent = `Updated: ${new Date().toLocaleTimeString()}`;

        appendPoint(new Date(), price);
        if (cashInput.value) runCashCalc();
      } catch (_) {
        updatedAt.textContent = "Update failed — retrying…";
      }
    }

    // Boot
    (async function init() {
      await seedChart();
      await tick();
      setInterval(tick, TICK_MS);
    })();
  </script>
</body>
</html>
export default {
  async fetch(request, env) {


