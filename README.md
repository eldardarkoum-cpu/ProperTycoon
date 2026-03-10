# ProperTycoon
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ProperTycoon - ملك العقارات</title>
    <style>
        :root {
            --primary-color: #0088cc;
            --bg-color: #1c1c1c;
            --card-bg: #2d2d2d;
            --text-color: #ffffff;
            --accent-color: #f39c12;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            user-select: none;
        }

        header {
            background-color: var(--primary-color);
            width: 100%;
            padding: 20px;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .stats-container {
            padding: 20px;
            text-align: center;
        }

        .balance {
            font-size: 2.5rem;
            font-weight: bold;
            color: var(--accent-color);
        }

        .income-rate {
            font-size: 1rem;
            opacity: 0.8;
        }

        .click-area {
            background: radial-gradient(circle, #3498db, #2980b9);
            width: 180px;
            height: 180px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 5rem;
            cursor: pointer;
            transition: transform 0.1s;
            margin: 20px 0;
            box-shadow: 0 10px 20px rgba(0,0,0,0.4);
        }

        .click-area:active {
            transform: scale(0.95);
        }

        .buildings-list {
            width: 90%;
            max-width: 500px;
            padding-bottom: 50px;
        }

        .building-card {
            background-color: var(--card-bg);
            border-radius: 12px;
            padding: 15px;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            border-right: 5px solid var(--primary-color);
        }

        .building-info {
            flex-grow: 1;
            margin-right: 15px;
        }

        .building-name {
            font-size: 1.2rem;
            font-weight: bold;
        }

        .building-cost {
            color: #2ecc71;
            font-weight: bold;
        }

        .upgrade-btn {
            background-color: var(--primary-color);
            border: none;
            color: white;
            padding: 10px 15px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }

        .upgrade-btn:disabled {
            background-color: #555;
            cursor: not-allowed;
        }

        .multiplier-badge {
            background: var(--accent-color);
            font-size: 0.8rem;
            padding: 2px 6px;
            border-radius: 4px;
            margin-right: 5px;
        }
    </style>
</head>
<body>

<header>
    <h1>🏢 ProperTycoon</h1>
</header>

<div class="stats-container">
    <div class="balance" id="balance">0 $</div>
    <div class="income-rate">الدخل لكل نقرة: <span id="click-power">1</span> $</div>
</div>

<div class="click-area" id="clicker">
    🏢
</div>

<div class="buildings-list" id="buildings">
    </div>

<script>
    let balance = 0;
    let clickPower = 1;

    const buildingsData = [
        { id: 1, name: "شقة استوديو", baseCost: 10, icon: "🏠", multiplier: 1 },
        { id: 2, name: "شقة عائلية", baseCost: 100, icon: "🏡", multiplier: 2 },
        { id: 3, name: "فيلا ريفية", baseCost: 500, icon: "🏘️", multiplier: 3 },
        { id: 4, name: "عمارة سكنية", baseCost: 2000, icon: "🏢", multiplier: 2 },
        { id: 5, name: "برج سكني", baseCost: 10000, icon: "🏙️", multiplier: 3 },
        { id: 6, name: "مجمع تجاري", baseCost: 50000, icon: "🏬", multiplier: 2 },
        { id: 7, name: "ناطحة سحاب صغيرة", baseCost: 200000, icon: "🏗️", multiplier: 3 },
        { id: 8, name: "مركز أعمال دولي", baseCost: 1000000, icon: "🏛️", multiplier: 2 },
        { id: 9, name: "برج خليفة المصغر", baseCost: 5000000, icon: "🗼", multiplier: 3 },
        { id: 10, name: "فندق ملكي فخم", baseCost: 20000000, icon: "🏰", multiplier: 5 }
    ];

    let ownedLevels = Array(10).fill(0);

    const balanceEl = document.getElementById('balance');
    const clickPowerEl = document.getElementById('click-power');
    const buildingsListEl = document.getElementById('buildings');
    const clickerEl = document.getElementById('clicker');

    function updateUI() {
        balanceEl.innerText = Math.floor(balance).toLocaleString() + " $";
        clickPowerEl.innerText = clickPower.toLocaleString();
        
        // Update buttons status
        buildingsData.forEach((b, index) => {
            const btn = document.getElementById(`btn-${index}`);
            if (btn) {
                btn.disabled = balance < b.baseCost;
            }
        });
    }

    function createBuildings() {
        buildingsData.forEach((b, index) => {
            const card = document.createElement('div');
            card.className = 'building-card';
            card.innerHTML = `
                <div style="font-size: 2rem; margin-left:15px;">${b.icon}</div>
                <div class="building-info">
                    <div class="building-name">${b.name} <span class="multiplier-badge">x${b.multiplier}</span></div>
                    <div class="building-cost">التكلفة: ${b.baseCost.toLocaleString()} $</div>
                </div>
                <button class="upgrade-btn" id="btn-${index}" onclick="buyBuilding(${index})">شراء</button>
            `;
            buildingsListEl.appendChild(card);
        });
    }

    function buyBuilding(index) {
        const b = buildingsData[index];
        if (balance >= b.baseCost) {
            balance -= b.baseCost;
            // تطبيق منطق المضاعفات
            clickPower *= b.multiplier;
            // زيادة التكلفة للمرة القادمة (اختياري لجعل اللعبة أصعب)
            buildingsData[index].baseCost = Math.floor(buildingsData[index].baseCost * 1.5);
            
            // تحديث واجهة المبنى
            const card = document.getElementsByClassName('building-card')[index];
            card.querySelector('.building-cost').innerText = `التكلفة: ${buildingsData[index].baseCost.toLocaleString()} $`;
            
            updateUI();
        }
    }

    clickerEl.addEventListener('click', () => {
        balance += clickPower;
        updateUI();
        
        // Animation effect
        clickerEl.style.transform = "scale(0.95)";
        setTimeout(() => clickerEl.style.transfor m = "scale(1)", 100);
    });

    // Initialize
    createBuildings();
    updateUI();
</script>

</body>
</html>
