<!DOCTYPE html>
<html>
<head>
    <title>منصة المذاكرة الجماعية</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f5f5f5;
        }
        .container {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            text-align: center;
        }
        input, button, select {
            padding: 10px;
            margin: 10px 0;
            width: 100%;
            border-radius: 5px;
            border: 1px solid #ddd;
        }
        button {
            background-color: #007BFF;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        .chat-box, .file-box {
            margin-top: 20px;
            border: 1px solid #ddd;
            padding: 10px;
            border-radius: 5px;
            max-height: 200px;
            overflow-y: auto;
            text-align: left;
        }
        .notification {
            color: green;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>منصة المذاكرة الجماعية</h1>
        <form id="study-form">
            <input type="text" id="name" placeholder="الاسم" required>
            <input type="text" id="subject" placeholder="المادة الدراسية" required>
            <button type="submit">انضم إلى مجموعة المذاكرة</button>
        </form>
        <div id="group-info"></div>
        <div class="notification" id="notification"></div>
        <div class="chat-box" id="chat-box"></div>
        <form id="chat-form" style="display:none;">
            <input type="text" id="chat-message" placeholder="اكتب رسالة..." required>
            <button type="submit">إرسال</button>
        </form>
        <div class="file-box" id="file-box"></div>
        <form id="file-form" style="display:none;" enctype="multipart/form-data">
            <input type="file" id="file-upload" required>
            <button type="submit">مشاركة الملف</button>
        </form>
        <h3>إعدادات الإشعارات</h3>
        <select id="notification-sound">
            <option value="sound1">صوت 1</option>
            <option value="sound2">صوت 2</option>
            <option value="sound3">صوت 3</option>
        </select>
    </div>

    <audio id="sound1" src="https://www.soundjay.com/button/beep-07.wav"></audio>
    <audio id="sound2" src="https://www.soundjay.com/button/beep-08b.wav"></audio>
    <audio id="sound3" src="https://www.soundjay.com/button/beep-09.wav"></audio>

    <script>
        let selectedSound = 'sound1';

        document.getElementById('notification-sound').addEventListener('change', function() {
            selectedSound = this.value;
        });

        document.getElementById('study-form').onsubmit = function(event) {
            event.preventDefault();
            const name = document.getElementById('name').value;
            const subject = document.getElementById('subject').value;
            document.getElementById('group-info').innerHTML = `<h3>مرحبًا ${name}!</h3><p>لقد انضممت إلى مجموعة مذاكرة مادة ${subject}.</p>`;
            document.getElementById('notification').innerText = 'تمت إضافة موعد المذاكرة بنجاح!';
            document.getElementById('chat-form').style.display = 'block';
            document.getElementById('file-form').style.display = 'block';
            sendNotification('مرحبًا بك في مجموعة المذاكرة!');
        }

        document.getElementById('chat-form').onsubmit = function(event) {
            event.preventDefault();
            const message = document.getElementById('chat-message').value;
            saveToNotion(message);
            document.getElementById('chat-message').value = '';
        }

        async function saveToNotion(message) {
            const response = await fetch('https://api.notion.com/v1/pages', {
                method: 'POST',
                headers: {
                    'Authorization': 'Bearer YOUR_NOTION_API_KEY',
                    'Content-Type': 'application/json',
                    'Notion-Version': '2021-08-16'
                },
                body: JSON.stringify({
                    parent: { database_id: 'YOUR_DATABASE_ID' },
                    properties: {
                        Name: {
                            title: [
                                {
                                    text: { content: message }
                                }
                            ]
                        }
                    }
                })
            });

            if (response.ok) {
                document.getElementById('chat-box').innerHTML += `<p>${message}</p>`;
                sendNotification('رسالة جديدة في مجموعة المذاكرة!');
            } else {
                console.error('Failed to save message to Notion');
            }
        }

        function sendNotification(message) {
            if (Notification.permission === "granted") {
                new Notification(message);
                playNotificationSound();
            } else if (Notification.permission !== "denied") {
                Notification.requestPermission().then(permission => {
                    if (permission === "granted") {
                        new Notification(message);
                        playNotificationSound();
                    }
                });
            }
        }

        function playNotificationSound() {
            document.getElementById(selectedSound).play();
        }
    </script>
</body>
</html>
