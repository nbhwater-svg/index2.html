<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WPM カウンター & タイマー</title>
    <style>
        :root {
            --primary-color: #1a73e8;
            --success-color: #188038;
            --danger-color: #d93025;
            --bg-color: #f8f9fa;
            --card-bg: #ffffff;
            --text-color: #202124;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 16px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 500px;
            background: var(--card-bg);
            padding: 24px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.08);
            box-sizing: border-box;
        }

        h1 {
            font-size: 20px;
            margin-top: 0;
            margin-bottom: 16px;
            text-align: center;
            color: var(--primary-color);
        }

        textarea {
            width: 100%;
            height: 120px;
            padding: 12px;
            border: 2px solid #dadce0;
            border-radius: 8px;
            font-size: 16px;
            resize: vertical;
            box-sizing: border-box;
            margin-bottom: 16px;
        }

        textarea:focus {
            border-color: var(--primary-color);
            outline: none;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            margin-bottom: 20px;
        }

        .stat-card {
            background: #f1f3f4;
            padding: 12px 4px;
            border-radius: 8px;
            text-align: center;
        }

        .stat-label {
            font-size: 12px;
            color: #5f6368;
            margin-bottom: 4px;
        }

        .stat-value {
            font-size: 20px;
            font-weight: bold;
            color: #1a73e8;
        }

        .btn {
            width: 100%;
            padding: 14px;
            font-size: 16px;
            font-weight: bold;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.2s;
            margin-bottom: 12px;
        }

        .btn-start {
            background-color: var(--primary-color);
            color: white;
        }

        .btn-start:hover { background-color: #1557b0; }

        .btn-stop {
            background-color: var(--success-color);
            color: white;
            display: none;
        }

        .btn-stop:hover { background-color: #13652b; }

        .btn-reset {
            background-color: #e8eaed;
            color: var(--text-color);
        }

        .btn-reset:hover { background-color: #dadce0; }
    </style>
</head>
<body>

<div class="container">
    <h1>WPM測定タイマー</h1>
    
    <textarea id="textInput" placeholder="ここに英語の文章を貼り付けてください..."></textarea>
    
    <div class="stats-grid">
        <div class="stat-card">
            <div class="stat-label">単語数 (Words)</div>
            <div id="wordCount" class="stat-value">0</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">時間 (Time)</div>
            <div id="timerDisplay" class="stat-value">00:00</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">WPM</div>
            <div id="wpmValue" class="stat-value" style="color: var(--success-color);">0</div>
        </div>
    </div>

    <button id="startBtn" class="btn btn-start">読書を開始する (Start)</button>
    <button id="stopBtn" class="btn btn-stop">読み終わった！ (Stop)</button>
    <button id="resetBtn" class="btn btn-reset">リセット (Reset)</button>
</div>

<script>
    let timerInterval = null;
    let startTime = 0;
    let elapsedTime = 0; // 単位: ミリ秒
    let wordCount = 0;

    const textInput = document.getElementById('textInput');
    const wordCountDisplay = document.getElementById('wordCount');
    const timerDisplay = document.getElementById('timerDisplay');
    const wpmDisplay = document.getElementById('wpmValue');
    
    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const resetBtn = document.getElementById('resetBtn');

    // 単語数をリアルタイムでカウントする関数
    function updateWordCount() {
        const text = textInput.value.trim();
        if (text === '') {
            wordCount = 0;
        } else {
            // 改行や連続するスペースを考慮して半角スペースで区切る
            wordCount = text.replace(/[\n\r]/g, ' ').split(/\s+/).filter(word => word.length > 0).length;
        }
        wordCountDisplay.textContent = wordCount;
        calculateWPM();
    }

    // WPMを計算する関数
    function calculateWPM() {
        if (elapsedTime === 0 || wordCount === 0) {
            wpmDisplay.textContent = 0;
            return;
        }
        const minutes = elapsedTime / 60000; // ミリ秒を分に変換
        const wpm = Math.round(wordCount / minutes);
        wpmDisplay.textContent = wpm;
    }

    // タイマー表示を更新する関数
    function updateTimerDisplay() {
        const totalSeconds = Math.floor(elapsedTime / 1000);
        const minutes = Math.floor(totalSeconds / 60);
        const seconds = totalSeconds % 60;
        
        const displayMin = String(minutes).padStart(2, '0');
        const displaySec = String(seconds).padStart(2, '0');
        timerDisplay.textContent = `${displayMin}:${displaySec}`;
    }

    // 入力エリアの変更イベント
    textInput.addEventListener('input', updateWordCount);

    // スタートボタン
    startBtn.addEventListener('click', () => {
        if (wordCount === 0) {
            alert('最初に英語の文章を入力するか、貼り付けてください。');
            return;
        }
        textInput.disabled = true; // 測定中は編集不可にする
        startBtn.style.display = 'none';
        stopBtn.style.display = 'block';
        
        startTime = Date.now() - elapsedTime;
        timerInterval = setInterval(() => {
            elapsedTime = Date.now() - startTime;
            updateTimerDisplay();
            calculateWPM(); // リアルタイムでWPMも更新
        }, 200);
    });

    // ストップボタン
    stopBtn.addEventListener('click', () => {
        clearInterval(timerInterval);
        stopBtn.style.display = 'none';
        startBtn.style.display = 'block';
        startBtn.textContent = '読書を再開する (Resume)';
        calculateWPM();
    });

    // リセットボタン
    resetBtn.addEventListener('click', () => {
        clearInterval(timerInterval);
        textInput.disabled = false;
        textInput.value = '';
        elapsedTime = 0;
        wordCount = 0;
        
        wordCountDisplay.textContent = '0';
        timerDisplay.textContent = '00:00';
        wpmDisplay.textContent = '0';
        
        stopBtn.style.display = 'none';
        startBtn.style.display = 'block';
        startBtn.textContent = '読書を開始する (Start)';
    });
</script>

</body>
</html>
