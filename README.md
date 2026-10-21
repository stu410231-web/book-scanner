<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📚 書本封面掃描登記系統</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }
        body {
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .card {
            background: white;
            padding: 30px 20px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
            width: 100%;
            max-width: 420px;
            text-align: center;
        }
        h2 { margin-top: 0; color: #2c3e50; font-size: 22px; }
        p { color: #666; font-size: 14px; margin-bottom: 25px; }
        .upload-btn {
            background-color: #4f46e5;
            color: white;
            padding: 14px 24px;
            border-radius: 10px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            display: inline-block;
            transition: all 0.2s ease;
            box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
        }
        .upload-btn:hover { background-color: #4338ca; transform: translateY(-2px); }
        input[type="file"] { display: none; }
        #preview {
            max-width: 100%;
            max-height: 250px;
            margin-top: 20px;
            border-radius: 10px;
            display: none;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        #status {
            margin-top: 20px;
            padding: 12px;
            border-radius: 8px;
            font-size: 15px;
            font-weight: 500;
            display: none;
        }
        .loading { background-color: #e0f2fe; color: #0369a1; }
        .success { background-color: #dcfce7; color: #15803d; }
        .error { background-color: #fee2e2; color: #b91c1c; }
    </style>
</head>
<body>

<div class="card">
    <h2>📖 拍照記錄書名</h2>
    <p>拍攝或上傳書本封面照片，系統將自動辨識書名並儲存至 Google 試算表。</p>

    <label class="upload-btn">
        📷 拍攝 / 上傳照片
        <input type="file" id="imageInput" accept="image/*" capture="environment">
    </label>

    <br>
    <img id="preview" alt="照片預覽">
    <div id="status"></div>
</div>

<script>
    // 已自動填入你提供的 Apps Script Web App 網址
    const GAS_WEB_APP_URL = "https://script.google.com/macros/s/AKfycbyGvBDQPmp03T8Eb-yKmNpUsYwDMSCdYk0HPHjdYfU-Czl25szniZQS6MpiiyuYsl-bGA/exec";

    document.getElementById('imageInput').addEventListener('change', function(e) {
        const file = e.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        const preview = document.getElementById('preview');
        const status = document.getElementById('status');

        reader.onload = function(event) {
            preview.src = event.target.result;
            preview.style.display = "inline-block";
            showStatus("⏳ AI 辨識中並儲存至試算表，請稍候...", "loading");

            const base64Data = event.target.result.split(',')[1];

            fetch(GAS_WEB_APP_URL, {
                method: 'POST',
                body: JSON.stringify({ image: base64Data })
            })
            .then(res => res.json())
            .then(data => {
                if (data.status === 'success') {
                    showStatus(`✅ 成功寫入！<br><b>《${data.title}》</b>`, "success");
                } else {
                    showStatus("❌ 辨識失敗：" + data.message, "error");
                }
            })
            .catch(err => {
                showStatus("❌ 發生網路錯誤，請確認 Apps Script 權限已設為所有人。", "error");
                console.error(err);
            });
        };

        reader.readAsDataURL(file);
    });

    function showStatus(text, className) {
        const status = document.getElementById('status');
        status.innerHTML = text;
        status.className = className;
        status.style.display = "block";
    }
</script>

</body>
</html>
