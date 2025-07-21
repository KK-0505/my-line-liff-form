<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>活動報名</title>
    <style>
        body { font-family: sans-serif; padding: 20px; }
        input { width: 100%; padding: 8px; margin-bottom: 10px; box-sizing: border-box; }
        button { width: 100%; padding: 10px; background-color: #00B900; color: white; border: none; border-radius: 5px; cursor: pointer; }
        #message { margin-top: 15px; text-align: center; }
    </style>
</head>
<body>
    <h2>活動報名表單</h2>
    <form id="registrationForm">
        <input type="hidden" id="uid" name="uid">

        <label for="memberId">會員編號：</label>
        <input type="text" id="memberId" name="memberId" placeholder="請輸入您的會員編號" required>
        <label for="name">姓名：</label>
        <input type="text" id="name" name="name" required>

        <label for="phone">電話：</label>
        <input type="tel" id="phone" name="phone" required>

        <button type="submit">確認送出</button>
    </form>
    <p id="message"></p>

    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <script>
        const GAS_WEB_APP_URL = 'YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL'; // <--- 請換成你的 GAS 網址

        async function main() {
            // 1. 初始化 LIFF
            await liff.init({ liffId: "YOUR_LIFF_ID" }); // <--- 請換成你的 LIFF ID

            if (!liff.isLoggedIn()) {
                liff.login();
                return;
            }

            // 2. 從 URL 取得 UID
            const urlParams = new URLSearchParams(window.location.search);
            const uid = urlParams.get('uid');
            if (uid) {
                document.getElementById('uid').value = uid;
            } else {
                document.getElementById('message').innerText = '錯誤：無法取得使用者資訊。';
                return;
            }
            
            // 3. (可選) 取得 LINE Profile 來預填姓名
            const profile = await liff.getProfile();
            document.getElementById('name').value = profile.displayName;
        }

        main();

        // 4. 監聽表單提交事件
        document.getElementById('registrationForm').addEventListener('submit', async (e) => {
            e.preventDefault(); // 防止頁面跳轉

            const submitButton = e.target.querySelector('button');
            submitButton.disabled = true;
            submitButton.innerText = '傳送中...';

            // ===== 修改此處，增加 memberId 到要提交的資料中 =====
            const formData = {
                uid: document.getElementById('uid').value,
                memberId: document.getElementById('memberId').value, // 新增這一行
                name: document.getElementById('name').value,
                phone: document.getElementById('phone').value,
            };
            // ===============================================

            try {
                // 5. 使用 fetch API 將資料送到 GAS
                const response = await fetch(GAS_WEB_APP_URL, {
                    method: 'POST',
                    mode: 'no-cors', // 因為 GAS 回傳的 Header 限制，通常用 no-cors
                    cache: 'no-cache',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(formData)
                });
                
                // 雖然 no-cors 模式下無法讀取 response 內容，但請求已發出
                document.getElementById('message').innerText = '報名成功！視窗即將關閉。';

                // 6. 延遲一下，然後關閉 LIFF 視窗
                setTimeout(() => {
                    liff.closeWindow();
                }, 2000);

            } catch (error) {
                document.getElementById('message').innerText = `發生錯誤：${error.message}`;
                submitButton.disabled = false;
                submitButton.innerText = '確認送出';
            }
        });
    </script>
</body>
</html>
