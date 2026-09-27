
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1分間英語スピーチトレーナー</title>
    <style>
        :root {
            --primary: #1a73e8;
            --accent: #d93025;
            --success: #188038;
            --bg: #f8f9fa;
            --card: #ffffff;
            --text: #202124;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 16px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 500px;
            background: var(--card);
            padding: 24px;
            border-radius: 16px;
            box-shadow: 0 4px 16px rgba(0,0,0,0.08);
            box-sizing: border-box;
            text-align: center;
        }

        h1 {
            font-size: 20px;
            margin-top: 0;
            color: var(--primary);
            margin-bottom: 20px;
        }

        /* タイマーのデザイン */
        .timer-circle {
            width: 140px;
            height: 140px;
            border-radius: 50%;
            border: 6px solid #e8eaed;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 auto 24px;
            position: relative;
            transition: border-color 0.3s;
        }

        .timer-circle.active {
            border-color: var(--accent);
        }

        .timer-display {
            font-size: 36px;
            font-weight: bold;
            font-variant-numeric: tabular-nums;
        }

        /* 状態表示ラベル */
        .status-badge {
            display: inline-block;
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: bold;
            background: #e8eaed;
            margin-bottom: 24px;
        }

        .status-badge.recording {
            background: #fce8e6;
            color: var(--accent);
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.6; }
            100% { opacity: 1; }
        }

        /* ボタン */
        .btn {
            width: 100%;
            padding: 14px;
            font-size: 16px;
            font-weight: bold;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            margin-bottom: 12px;
            transition: background 0.2s, transform 0.1s;
        }

        .btn:active { transform: scale(0.98); }

        .btn-start { background-color: var(--primary); color: white; }
        .btn-start:hover { background-color: #1557b0; }

        .btn-stop { background-color: var(--accent); color: white; display: none; }
        .btn-stop:hover { background-color: #b31412; }

        /* オーディオプレイヤー */
        .audio-section {
            margin-top: 20px;
            padding-top: 20px;
            border-top: 1px solid #e8eaed;
            display: none;
        }

        .audio-section h3 {
            font-size: 15px;
            margin: 0 0 10px;
            color: #5f6368;
            text-align: left;
        }

        audio { width: 100%; margin-bottom: 10px; }

        /* 文字起こし結果のエリア */
        .transcript-section {
            margin-top: 20px;
            text-align: left;
        }

        .transcript-section h3 {
            font-size: 15px;
            margin: 0 0 10px;
            color: #5f6368;
        }

        .transcript-box {
            width: 100%;
            min-height: 100px;
            max-height: 200px;
            overflow-y: auto;
            padding: 12px;
            background: #f1f3f4;
            border-radius: 8px;
            font-size: 15px;
            line-height: 1.5;
            box-sizing: border-box;
            white-space: pre-wrap;
            border: 1px solid #dadce0;
        }

        .placeholder-text {
            color: #70757a;
            font-style: italic;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>1分間英語スピーチトレーナー</h1>
    
    <div id="timerCircle" class="timer-circle">
        <div id="timerDisplay" class="timer-display">01:00</div>
    </div>

    <div>
        <div id="statusBadge" class="status-badge">準備完了</div>
    </div>

    <button id="startBtn" class="btn btn-start">スピーチを開始する (Start)</button>
    <button id="stopBtn" class="btn btn-stop">途中で終了する (Stop)</button>

    <!-- 録音再生セクション -->
    <div id="audioSection" class="audio-section">
        <h3>録音された音声 (Playback):</h3>
        <audio id="audioPlayer" controls></audio>
    </div>

    <!-- 文字起こしセクション -->
    <div class="transcript-section">
        <h3>文字起こし結果 (English Transcript):</h3>
        <div id="transcriptBox" class="transcript-box">
            <span class="placeholder-text">ここにあなたのスピーチした英語がリアルタイムで文字起こしされます...</span>
        </div>
    </div>
</div>

<script>
    let countdownInterval = null;
    let timeLeft = 60; // 1分間 (60秒)

    // 録音関連の変数
    let mediaRecorder = null;
    let audioChunks = [];

    // 音声認識（文字起こし）関連の変数
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    let recognition = null;

    if (SpeechRecognition) {
        recognition = new SpeechRecognition();
        recognition.continuous = true;       // 途切れても録音を続ける
        recognition.interimResults = true;    // 途中の経過も表示する
        recognition.lang = 'en-US';           // 英語（米国）に設定
    }

    const timerDisplay = document.getElementById('timerDisplay');
    const timerCircle = document.getElementById('timerCircle');
    const statusBadge = document.getElementById('statusBadge');
    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const audioSection = document.getElementById('audioSection');
    const audioPlayer = document.getElementById('audioPlayer');
    const transcriptBox = document.getElementById('transcriptBox');

    // 1分間のタイマーを管理する関数
    function startTimer() {
        timeLeft = 60;
        updateTimerDisplay();
        timerCircle.classList.add('active');

        countdownInterval = setInterval(() => {
            timeLeft--;
            updateTimerDisplay();

            if (timeLeft <= 0) {
                endSpeech(true); // 1分経ったら自動終了
            }
        }, 1000);
    }

    function updateTimerDisplay() {
        const minutes = Math.floor(timeLeft / 60);
        const seconds = timeLeft % 60;
        timerDisplay.textContent = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    }

    // スピーチ（録音＆文字起こし）の開始
    startBtn.addEventListener('click', async () => {
        // マイク権限の取得と録音の準備
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            mediaRecorder = new MediaRecorder(stream);
            audioChunks = [];

            mediaRecorder.ondataavailable = (event) => {
                audioChunks.push(event.data);
            };

            mediaRecorder.onstop = () => {
                const audioBlob = new Blob(audioChunks, { type: 'audio/mp3' });
                const audioUrl = URL.createObjectURL(audioBlob);
                audioPlayer.src = audioUrl;
                audioSection.style.display = 'block';
                
                // マイクストリームを停止して解放
                stream.getTracks().forEach(track => track.stop());
            };

            // UI切り替え
            startBtn.style.display = 'none';
            stopBtn.style.display = 'block';
            statusBadge.textContent = '録音＆文字起こし中...';
            statusBadge.classList.add('recording');
            transcriptBox.innerHTML = ''; // ボックスをクリア

            // 録音と文字起こしのスタート
            mediaRecorder.start();
            if (recognition) {
                recognition.start();
            } else {
                transcriptBox.innerHTML = '<span class="placeholder-text" style="color:red;">お使いのブラウザは文字起こしに対応していません。(PCのGoogle Chromeを推奨します)</span>';
            }

            startTimer();

        } catch (err) {
            alert('マイクのアクセスが拒否されたか、マイクが見つかりません。設定を確認してください。');
            console.error(err);
        }
    });

    // 文字起こしのリアルタイム処理
    if (recognition) {
        recognition.onresult = (event) => {
            let interimTranscript = '';
            let finalTranscript = '';

            for (let i = event.resultIndex; i < event.results.length; ++i) {
                if (event.results[i].isFinal) {
                    finalTranscript += event.results[i][0].transcript + ' ';
                } else {
                    interimTranscript += event.results[i][0].transcript;
                }
            }
            // 画面に文字起こし結果を反映
            transcriptBox.innerHTML = `<strong>${finalTranscript}</strong><span style="color: #70757a;">${interimTranscript}</span>`;
            
            // 常に最下部まで自動スクロール
            transcriptBox.scrollTop = transcriptBox.scrollHeight;
        };

        recognition.onerror = (event) => {
            console.error('Speech recognition error', event.error);
        };
    }

    // スピーチの終了処理
    stopBtn.addEventListener('click', () => {
        endSpeech(false);
    });

    function endSpeech(isTimeUp) {
        clearInterval(countdownInterval);
        timerCircle.classList.remove('active');

        // 録音と文字起こしの停止
        if (mediaRecorder && mediaRecorder.state !== 'inactive') {
            mediaRecorder.stop();
        }
        if (recognition) {
            recognition.stop();
        }

        // UIの復元
        stopBtn.style.display = 'none';
        startBtn.style.display = 'block';
        startBtn.textContent = 'もう一度挑戦する (Restart)';
        
        statusBadge.classList.remove('recording');
        statusBadge.textContent = isTimeUp ? 'タイムアップ！終了しました' : '途中で停止しました';
    }
</script>

</body>
