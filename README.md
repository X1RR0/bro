<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Інтерактивна карта областей України — X1RR0</title>
    <link rel="stylesheet" href="https://unpkg.com" />
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        }
        body {
            background-color: #000000;
            color: #f8fafc;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            overflow-x: hidden;
        }

        /* Чорний вступний екран */
        #intro-screen {
            position: fixed;
            inset: 0;
            background-color: #000000;
            z-index: 9000;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 24px;
            text-align: center;
            transition: opacity 0.8s ease;
        }
        #intro-screen.hidden {
            opacity: 0;
            pointer-events: none;
        }
        #intro-text {
            font-size: clamp(1.5rem, 5vw, 2.8rem);
            font-weight: 700;
            color: #ffffff;
            letter-spacing: 0.5px;
            opacity: 0;
            filter: blur(0px);
            transform: scale(0.95);
            transition: opacity 1.8s ease, filter 1.6s ease, transform 1.8s ease;
        }
        #intro-text.visible {
            opacity: 1;
            transform: scale(1);
        }
        #intro-text.blur-out {
            opacity: 0;
            filter: blur(18px);
            transform: scale(1.08);
        }

        /* Кнопка пропуску анімацій */
        #skip-btn {
            position: fixed;
            bottom: 24px;
            right: 20px;
            z-index: 9999;
            padding: 10px 18px;
            background: rgba(30, 41, 59, 0.9);
            color: #f8fafc;
            border: 1px solid #475569;
            border-radius: 999px;
            font-size: 0.88rem;
            font-weight: 600;
            cursor: pointer;
            backdrop-filter: blur(6px);
            transition: background 0.2s, transform 0.15s;
        }
        #skip-btn:hover {
            background: #334155;
            transform: scale(1.04);
        }
        #skip-btn.hidden {
            display: none;
        }

        /* Основний інтерфейс */
        header {
            background-color: #0f172a;
            padding: 14px 16px;
            text-align: center;
            border-bottom: 1px solid #1e293b;
        }
        header h1 {
            font-size: 1.25rem;
            margin-bottom: 4px;
        }
        header p {
            font-size: 0.85rem;
            color: #94a3b8;
        }
        .legend {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            align-items: center;
            gap: 8px;
            padding: 10px;
            background-color: #0f172a;
            border-bottom: 1px solid #1e293b;
            font-size: 0.78rem;
        }
        .step-badge {
            display: inline-flex;
            align-items: center;
            gap: 5px;
            background-color: #1e293b;
            padding: 4px 8px;
            border-radius: 6px;
            border: 1px solid #334155;
        }
        .dot {
            width: 11px;
            height: 11px;
            border-radius: 50%;
            display: inline-block;
        }
        .reset-btn {
            padding: 6px 12px;
            border: none;
            border-radius: 6px;
            background-color: #334155;
            color: #f8fafc;
            font-size: 0.8rem;
            cursor: pointer;
        }
        .reset-btn:hover {
            background-color: #475569;
        }
        #map-wrapper {
            flex: 1;
            position: relative;
            width: 100%;
            height: 68vh;
            min-height: 460px;
            background-color: #000000;
        }
        #map {
            width: 100%;
            height: 100%;
            background-color: #000000;
        }
        #info-panel {
            position: absolute;
            bottom: 18px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 1000;
            background: rgba(15, 23, 42, 0.92);
            padding: 10px 16px;
            border-radius: 10px;
            border: 1px solid #475569;
            font-size: 0.9rem;
            text-align: center;
            pointer-events: none;
            max-width: 90%;
        }
        footer {
            background-color: #0f172a;
            padding: 12px 16px;
            text-align: center;
            font-size: 0.78rem;
            color: #94a3b8;
            border-top: 1px solid #1e293b;
        }
    </style>
</head>
<body>

<div id="intro-screen">
    <div id="intro-text">Інтерактивна карта України від :X1RR0</div>
</div>

<button id="skip-btn">Пропустити анімацію ⏭</button>

<header>
    <h1>🇺🇦 Інтерактивна карта України</h1>
    <p>Натискай на область, щоб змінювати її колір по черзі</p>
</header>

<div class="legend">
    <span class="step-badge"><span class="dot" style="background:#facc15"></span>1. Жовтий</span>
    <span class="step-badge"><span class="dot" style="background:#ef4444"></span>2. Червоний</span>
    <span class="step-badge"><span class="dot" style="background:#7f1d1d"></span>3. Темно-червоний</span>
    <span class="step-badge"><span class="dot" style="background:#f97316"></span>4. Оранжевий</span>
    <span class="step-badge"><span class="dot" style="background:#22c55e"></span>5. Зелений</span>
    <span class="step-badge"><span class="dot" style="background:#a855f7"></span>6. Фіолетовий</span>
    <span class="step-badge"><span class="dot" style="background:#a16207"></span>7. Темно-жовтий</span>
    <button class="reset-btn" id="reset-btn">Скинути все</button>
</div>

<div id="map-wrapper">
    <div id="map"></div>
    <div id="info-panel">Підготовка карти...</div>
</div>

<footer>
    ⚠️ Неофіційний розважальний проєкт. Створено виключно для розваги та особистого використання. Автор: <strong>X1RR0</strong>.
</footer>

<script src="https://unpkg.com"></script>
<script>
    const colorSequence = [
        { name: 'Початковий', hex: '#334155' },
        { name: 'Жовтий', hex: '#facc15' },
        { name: 'Червоний', hex: '#ef4444' },
        { name: 'Темно-червоний', hex: '#7f1d1d' },
        { name: 'Оранжевий', hex: '#f97316' },
        { name: 'Зелений', hex: '#22c55e' },
        { name: 'Фіолетовий', hex: '#a855f7' },
        { name: 'Темно-жовтий', hex: '#a16207' }
    ];

    const regionState = {};
    const regionLayers = [];
    let fullBounds = null;
    let isSkipped = false;
    let isAnimating = true;

    const introScreen = document.getElementById('intro-screen');
    const introText = document.getElementById('intro-text');
    const skipBtn = document.getElementById('skip-btn');
    const infoPanel = document.getElementById('info-panel');

    const map = L.map('map', {
        zoomControl: true,
        attributionControl: false
    }).setView([48.5, 31.2], 6);

    function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    function getRegionName(props) {
        return props.name_uk || props.NAME_1 || props.name || props.shapeName || 'Область України';
    }

    function setupLayerInteractions(layer, name) {
        layer.on({
            mouseover: function (e) {
                if (isAnimating) return;
                e.target.setStyle({
                    weight: 3,
                    color: '#ffffff'
                });
                const currentStep = regionState[name] || 0;
                infoPanel.textContent = `${name} (${colorSequence[currentStep].name})`;
            },
            mouseout: function (e) {
                if (isAnimating) return;
                const step = regionState[name] || 0;
                e.target.setStyle({
                    weight: 1.5,
                    color: '#94a3b8',
                    fillColor: colorSequence[step].hex,
                    fillOpacity: 0.88
                });
            },
            click: function (e) {
                if (isAnimating) return;
                const nextStep = ((regionState[name] || 0) + 1) % colorSequence.length;
                regionState[name] = nextStep;
                const chosen = colorSequence[nextStep];

                e.target.setStyle({
                    fillColor: chosen.hex,
                    fillOpacity: 0.92
                });
                infoPanel.textContent = `${name}: ${chosen.name} (${nextStep}/7)`;
            }
        });
    }

    function showAllImmediately() {
        isSkipped = true;
        isAnimating = false;
        introScreen.classList.add('hidden');
        skipBtn.classList.add('hidden');

        regionLayers.forEach(item => {
            if (!map.hasLayer(item.layer)) {
                item.layer.addTo(map);
            }
            const step = regionState[item.name] || 0;
            item.layer.setStyle({
                fillColor: colorSequence[step].hex,
                fillOpacity: 0.88,
                opacity: 1,
                weight: 1.5,
                color: '#94a3b8'
            });
        });

        if (fullBounds) {
            map.fitBounds(fullBounds, { animate: true, duration: 0.6 });
        }
        infoPanel.textContent = 'Натисніть на будь-яку область';
    }

    skipBtn.addEventListener('click', showAllImmediately);

    document.getElementById('reset-btn').addEventListener('click', () => {
        for (let key in regionState) {
            delete regionState[key];
        }
