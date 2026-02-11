# index.html<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Investiční Portál | Kalkulačka</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #00d1b2;
            --primary-dark: #00b89c;
            --accent: #4a69bd;
            --bg: #f0f2f5;
            --card-bg: #ffffff;
            --text-main: #2c3e50;
            --text-muted: #636e72;
        }

        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            color: var(--text-main);
            margin: 0;
            padding: 40px 20px;
            min-height: 100vh;
        }

        .container {
            max-width: 1100px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 40px;
        }

        header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            color: #1a1a1a;
        }

        .grid {
            display: grid;
            grid-template-columns: 380px 1fr;
            gap: 30px;
            align-items: start;
        }

        @media (max-width: 900px) {
            .grid { grid-template-columns: 1fr; }
        }

        .card {
            background: var(--card-bg);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.05);
        }

        .input-section h3 {
            margin-top: 25px;
            padding-top: 15px;
            border-top: 1px solid #eee;
            font-size: 1.1rem;
        }

        .input-group { margin-bottom: 18px; }

        label {
            display: block;
            font-size: 0.85rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            color: var(--text-muted);
            margin-bottom: 8px;
        }

        input, select {
            width: 100%;
            padding: 12px 15px;
            border: 2px solid #edf2f7;
            border-radius: 10px;
            font-size: 1rem;
            transition: border-color 0.2s;
            outline: none;
        }

        input:focus { border-color: var(--primary); }

        button {
            width: 100%;
            padding: 15px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 12px;
            font-size: 1.1rem;
            font-weight: 700;
            cursor: pointer;
            margin-top: 20px;
            transition: transform 0.2s, background 0.2s;
        }

        button:hover {
            background: var(--primary-dark);
            transform: translateY(-2px);
        }

        .results-summary {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .stat-box {
            padding: 20px;
            border-radius: 15px;
            background: #f8fafc;
            border: 1px solid #eef2f7;
        }

        .stat-box.highlight {
            background: #ebfefb;
            border-color: var(--primary);
        }

        .stat-label { font-size: 0.8rem; color: var(--text-muted); margin-bottom: 5px; }
        .stat-value { font-size: 1.4rem; font-weight: 800; color: #1a1a1a; }

        .chart-box {
            position: relative;
            height: 450px;
            width: 100%;
        }

        .badge {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.7rem;
            background: #eee;
            margin-left: 5px;
            vertical-align: middle;
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Investiční Laboratoř</h1>
        <p>Naplánujte si svou finanční svobodu s přesností na korunu.</p>
    </header>
    
    <div class="grid">
        <div class="card input-section">
            <div class="input-group">
                <label>Počáteční kapitál</label>
                <input type="number" id="initialDeposit" value="100000" placeholder="Kč">
            </div>
            <div class="input-group">
                <label>Měsíční úložka</label>
                <input type="number" id="monthlyContribution" value="5000" placeholder="Kč">
            </div>
            <div class="input-group">
                <label>Doba (roky)</label>
                <input type="number" id="years" value="25">
            </div>
            <div class="input-group">
                <label>Roční výnos <span class="badge">p.a.</span></label>
                <input type="number" id="annualReturn" value="8" step="0.1">
            </div>
            <div class="input-group">
                <label>Inflace <span class="badge">odhad</span></label>
                <input type="number" id="inflation" value="3" step="0.1">
            </div>
            <div class="input-group">
                <label>Připisování úroků</label>
                <select id="compoundFrequency">
                    <option value="12">Měsíčně (Standard)</option>
                    <option value="4">Kvartálně</option>
                    <option value="1">Ročně</option>
                </select>
            </div>

            <h3>Dynamické navyšování</h3>
            <div style="display: flex; gap: 10px;">
                <div class="input-group">
                    <label>Každých (let)</label>
                    <input type="number" id="stepYears" value="3">
                </div>
                <div class="input-group">
                    <label>Navýšit o (Kč)</label>
                    <input type="number" id="stepAmount" value="500">
                </div>
            </div>

            <button onclick="calculate()">Přepočítat portfolio</button>
        </div>

        <div class="card">
            <div class="results-summary">
                <div class="stat-box highlight">
                    <div class="stat-label">Cílová částka</div>
                    <div id="resTotal" class="stat-value">0 Kč</div>
                </div>
                <div class="stat-box">
                    <div class="stat-label">V dnešních cenách</div>
                    <div id="resReal" class="stat-value">0 Kč</div>
                </div>
                <div class="stat-box">
                    <div class="stat-label">Celkem vloženo</div>
                    <div id="resDeposits" class="stat-value">0 Kč</div>
                </div>
            </div>

            <div class="chart-box">
                <canvas id="growthChart"></canvas>
            </div>
        </div>
    </div>
</div>

<script>
    let myChart = null;

    function calculate() {
        const initialDeposit = parseFloat(document.getElementById('initialDeposit').value) || 0;
        let monthlyContribution = parseFloat(document.getElementById('monthlyContribution').value) || 0;
        const years = parseInt(document.getElementById('years').value) || 1;
        const annualReturn = parseFloat(document.getElementById('annualReturn').value) / 100 || 0;
        const inflation = parseFloat(document.getElementById('inflation').value) / 100 || 0;
        const freq = parseInt(document.getElementById('compoundFrequency').value);
        const stepYears = parseInt(document.getElementById('stepYears').value) || 0;
        const stepAmount = parseFloat(document.getElementById('stepAmount').value) || 0;

        let total = initialDeposit;
        let totalDeposits = initialDeposit;
        
        let historyTotal = [initialDeposit];
        let historyDeposits = [initialDeposit];
        let labels = [0];

        for (let m = 1; m <= years * 12; m++) {
            if (stepYears > 0 && m > 1 && (m - 1) % (stepYears * 12) === 0) {
                monthlyContribution += stepAmount;
            }

            total += monthlyContribution;
            totalDeposits += monthlyContribution;

            if (m % (12 / freq) === 0) {
                total *= (1 + (annualReturn / freq));
            }

            if (m % 12 === 0) {
                historyTotal.push(Math.round(total));
                historyDeposits.push(Math.round(totalDeposits));
                labels.push(m / 12);
            }
        }

        const realValue = total / Math.pow(1 + inflation, years);
        const formatter = new Intl.NumberFormat('cs-CZ', { style: 'currency', currency: 'CZK', maximumFractionDigits: 0 });
        
        document.getElementById('resTotal').innerText = formatter.format(total);
        document.getElementById('resReal').innerText = formatter.format(realValue);
        document.getElementById('resDeposits').innerText = formatter.format(totalDeposits);

        updateChart(labels, historyTotal, historyDeposits);
    }

    function updateChart(labels, dataTotal, dataDeposits) {
        const ctx = document.getElementById('growthChart').getContext('2d');
        if (myChart) myChart.destroy();

        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Celková hodnota (vč. úroků)',
                        data: dataTotal,
                        borderColor: '#00d1b2',
                        backgroundColor: 'rgba(0, 209, 178, 0.1)',
                        fill: true,
                        tension: 0.4,
                        pointRadius: 2
                    },
                    {
                        label: 'Vaše vklady',
                        data: dataDeposits,
                        borderColor: '#4a69bd',
                        borderDash: [5, 5],
                        fill: false,
                        tension: 0,
                        pointRadius: 0
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'bottom' }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        ticks: { callback: v => v.toLocaleString('cs-CZ') + ' Kč' }
                    }
                }
            }
        });
    }

    calculate();
</script>

</body>
</html>
