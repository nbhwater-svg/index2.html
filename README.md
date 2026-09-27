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
