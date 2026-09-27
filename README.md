<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>見やすい WPM カウンター & タイマー</title>
    <style>
        :root {
            --primary-color: #1a73e8;
            --success-color: #13652b;
            --danger-color: #d93025;
            --bg-color: #f8f9fa;
            --card-bg: #ffffff;
            --text-color: #202124;
            /* 読みやすさのためのカラー設定 */
            --textarea-text-color: #111111; /* しっかりと濃い黒色に変えて読みやすく */
            --textarea-bg-color: #ffffff;
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
            max-width: 550px;
            background: var(--card-bg);
            padding: 24px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.08);
            box-sizing: border-box;
        }

        h1 {
            font-size: 22px;
            margin-top: 0;
            margin-bottom: 16px;
            text-align: center;
            color: var(--primary-color);
        }

        /* 英文エリアの大幅なデザイン改善 */
        .input-label {
            font-size: 14px;
            font-weight: bold;
            color: #5f6368;
            margin-bottom: 6px;
            display: block;
            text-align: left;
        }

        textarea {
            width: 100%;
            height: 240px; /* 高さを2倍にして、長文も見やすく変更 */
            padding: 16px;
            border: 2px solid #9aa0a6; /* 枠線を少し太く、濃くして視認性アップ */
            border-radius: 8px;
            
            /* ★文字の読みやすさに特化した設定 */
            font-size: 18px; /* 文字サイズを大きく（16px → 18px） */
            line-height: 1.6; /* 行間を広げて、文章を読みやすく */
            color: var(--textarea-text-color); /* 濃い黒色に固定 */
            background-color: var(--textarea-bg-color);
            
            font-family: inherit;
            resize: vertical;
            box-sizing: border-box;
            margin-bottom: 18px;
        }

        textarea:focus {
            border-color: var(--primary-color);
            outline: none;
            background-color: #fff;
        }

        /* 読水（測定中）のとき、より集中できるようにするスタイル */
        textarea:disabled {
            background-color: #fdfdfd;
            color: #000000; /* 測定中のフォントも灰色にならず黒いまま維持 */
            border-color: #1a73e8;
            opacity: 1; /* スマホで薄くなる現象を防止 */
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
            border: 1px solid #dadce0;
        }

        .stat-label {
            font-size: 13px;
            font-weight: bold;
            color: #5f6368;
            margin-bottom: 4px;
        }

        .stat-value {
            font-size: 22px;
            font-weight: bold;
            color: #1a73e8;
        }

        .btn {
            width: 100%;
            padding: 16px; /* ボタンを少し大きくして押しやすく */
            font-size: 18px;
            font-weight: bold;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.2s;
            margin-bottom: 12px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .btn-start {
            background-color: var(--primary-color);
            color: white;
        }

        .btn-start:hover { background-color: #1557b0; }

        .btn-stop {
            background-color: #188038;
            color: white;
            display: none;
        }

        .btn-stop:hover { background-color: var(--success-color); }

        .btn-reset {
            background-color: #e8eaed;
            color: var(--text-color);
            font-size: 16px;
            padding: 12px;
        }

        .btn-reset:hover { background-color: #dadce0; }
    </style>
</head>
<body>

<div class="container">
    <h1>WPM測定タイマー (見易さ改善版)</h1>
    
    <label class="input-label" for="textInput">ここに英語の文章を貼り付けます：</label>
    <textarea id="textInput" placeholder="ここに英語の文章を貼り付けると、ハッキリとした黒い大きな文字で表示されます..."></textarea>
    
    <div class="stats-grid">
        <div class="stat-card">
            <div class="stat-label">単語数</div>
            <div id="wordCount" class="stat-value">0</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">経過時間</div>
            <div id="timerDisplay" class="stat-value">00:00</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">あなたのWPM</div>
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
    let elapsedTime = 0; 
    let wordCount = 0;

    const textInput = document.getElementById('textInput');
    const wordCountDisplay = document.getElementById('wordCount');
    const timerDisplay = document.getElementById('timerDisplay');
    const wpmDisplay = document.getElementById('wpmValue');
    
    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const resetBtn = document.getElementById('resetBtn');

    function updateWordCount() {
        const text = textInput.value.trim();
        if (text === '') {
            wordCount = 0;
        } else {
            wordCount = text.replace(/[\n\r]/g, ' ').split(/\s+/).filter(word => word.length > 0).length;
        }
        wordCountDisplay.textContent = wordCount;
        calculateWPM();
    }

    function calculateWPM() {
        if (elapsedTime === 0 || wordCount === 0) {
            wpmDisplay.textContent = 0;
            return;
        }
        const minutes = elapsedTime / 60000; 
        const wpm = Math.round(wordCount / minutes);
        wpmDisplay.textContent = wpm;
    }

    function updateTimerDisplay() {
        const totalSeconds = Math.floor(elapsedTime / 1000);
        const minutes = Math.floor(totalSeconds / 60);
        const seconds = totalSeconds % 60;
        
        const displayMin = String(minutes).padStart(2, '0');
        const displaySec = String(seconds).padStart(2, '0');
        timerDisplay.textContent = `${displayMin}:${displaySec}`;
    }

    textInput.addEventListener('input', updateWordCount);

    startBtn.addEventListener('click', () => {
        if (wordCount === 0) {
            alert('最初に英語の文章を入力するか、貼り付けてください。');
            return;
        }
        textInput.disabled = true; 
        startBtn.style.display = 'none';
        stopBtn.style.display = 'block';
        
        startTime = Date.now() - elapsedTime;
        timerInterval = setInterval(() => {
            elapsedTime = Date.now() - startTime;
            updateTimerDisplay();
            calculateWPM(); 
        }, 200);
    });

    stopBtn.addEventListener('click', () => {
        clearInterval(timerInterval);
        stopBtn.style.display = 'none';
        startBtn.style.display = 'block';
        startBtn.textContent = '読書を再開する (Resume)';
        calculateWPM();
    });

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
