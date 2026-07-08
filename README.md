<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Дневник Идей</title>
    <link rel="manifest" href="data:application/manifest+json,%7B%22name%22%3A%22%D0%94%D0%BD%D0%B5%D0%B2%D0%BD%D0%B8%D0%BA%20DeepSeek%22%2C%22short_name%22%3A%22%D0%94%D0%BD%D0%B5%D0%B2%D0%BD%D0%B8%D0%BA%22%2C%22start_url%22%3A%22.%2F%22%2C%22display%22%3A%22standalone%22%2C%22background_color%22%3A%22%23ffffff%22%2C%22theme_color%22%3A%22%231677ff%22%2C%22icons%22%3A%5B%7B%22src%22%3A%22data%3Aimage%2Fsvg%2Bxml%3Csvg%20xmlns%3D'http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg'%20viewBox%3D'0%200%20100%20100'%3E%3Crect%20width%3D'100'%20height%3D'100'%20rx%3D'20'%20fill%3D'%231677ff'%2F%3E%3Ctext%20x%3D'50'%20y%3D'65'%20text-anchor%3D'middle'%20font-size%3D'50'%20fill%3D'white'%3E📓%3C%2Ftext%3E%3C%2Fsvg%3E%22%2C%22sizes%22%3A%22100x100%22%2C%22type%22%3A%22image%2Fsvg%2Bxml%22%7D%5D%7D">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            margin: 0;
            background: #f5f5f5;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        .header {
            background: #1677ff;
            color: white;
            padding: 15px;
            text-align: center;
            font-size: 1.2em;
            font-weight: bold;
        }
        .chat {
            flex: 1;
            overflow-y: auto;
            padding: 10px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .message {
            max-width: 85%;
            padding: 12px;
            border-radius: 18px;
            line-height: 1.4;
            position: relative;
            word-wrap: break-word;
        }
        .user {
            align-self: flex-end;
            background: #dcf8c6;
            border-bottom-right-radius: 4px;
        }
        .assistant {
            align-self: flex-start;
            background: white;
            border-bottom-left-radius: 4px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
        .assistant b, .assistant strong {
            color: #1677ff;
        }
        .input-area {
            display: flex;
            padding: 10px;
            background: white;
            border-top: 1px solid #ddd;
            gap: 8px;
            align-items: center;
        }
        #messageInput {
            flex: 1;
            padding: 12px;
            border-radius: 24px;
            border: 1px solid #ccc;
            font-size: 16px;
            outline: none;
        }
        button {
            background: #1677ff;
            color: white;
            border: none;
            padding: 12px 18px;
            border-radius: 24px;
            font-size: 16px;
            cursor: pointer;
            white-space: nowrap;
        }
        .generate-btn {
            background: #ff9500;
            display: block;
            width: 90%;
            margin: 10px auto;
            text-align: center;
            font-weight: bold;
            padding: 14px;
            font-size: 18px;
        }
        .settings {
            background: #eee;
            padding: 10px;
            font-size: 14px;
            display: flex;
            align-items: center;
            gap: 8px;
            flex-wrap: wrap;
        }
        .settings input {
            flex: 2;
            min-width: 200px;
            padding: 8px;
            border-radius: 8px;
            border: 1px solid #ccc;
        }
        .settings button {
            background: #555;
        }
        .note {
            color: #666;
            font-size: 12px;
            padding: 0 10px 5px;
        }
    </style>
</head>
<body>
    <div class="header">📓 Дневник Идей</div>
    <div class="settings" id="settingsPanel">
        <span>🔑 API-ключ DeepSeek:</span>
        <input type="password" id="apiKeyInput" placeholder="sk-...">
        <button onclick="saveApiKey()">Сохранить</button>
        <span id="status" style="margin-left: auto; font-size: 12px;"></span>
    </div>
    <div class="chat" id="chatContainer"></div>
    <div class="note">💡 Отправляйте мысли текстом в течение дня. Вечером нажмите кнопку "Создать дневник".</div>
    <button class="generate-btn" onclick="generateDiary()">✨ Создать дневник за сегодня</button>
    <div class="input-area">
        <input type="text" id="messageInput" placeholder="Запиши свою мысль..." autocomplete="off">
        <button onclick="sendMessage()">📝</button>
        <button onclick="clearDay()" style="background:#ff4444;">🗑️</button>
    </div>

    <script>
        const STORAGE_KEY = 'diary_entries';
        const API_KEY_STORAGE = 'deepseek_api_key';

        let apiKey = localStorage.getItem(API_KEY_STORAGE) || '';
        document.getElementById('apiKeyInput').value = apiKey;

        function getTodayKey() {
            const d = new Date();
            return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
        }

        function getEntries() {
            const all = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
            const key = getTodayKey();
            return all[key] || [];
        }

        function saveEntries(entries) {
            const all = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
            const key = getTodayKey();
            all[key] = entries;
            localStorage.setItem(STORAGE_KEY, JSON.stringify(all));
        }

        function renderChat() {
            const container = document.getElementById('chatContainer');
            const entries = getEntries();
            container.innerHTML = '';
            entries.forEach(entry => {
                const msgDiv = document.createElement('div');
                msgDiv.className = 'message ' + (entry.role === 'user' ? 'user' : 'assistant');
                // Простая замена Markdown-жирности на HTML-теги
                let html = entry.content
                    .replace(/\*\*(.*?)\*\*/g, '<b>$1</b>')
                    .replace(/__(.*?)__/g, '<b>$1</b>')
                    .replace(/\n/g, '<br>');
                msgDiv.innerHTML = html;
                container.appendChild(msgDiv);
            });
            container.scrollTop = container.scrollHeight;
        }

        function addUserMessage(text) {
            const entries = getEntries();
            entries.push({ role: 'user', content: text });
            saveEntries(entries);
            renderChat();
        }

        function addAssistantMessage(text) {
            const entries = getEntries();
            entries.push({ role: 'assistant', content: text });
            saveEntries(entries);
            renderChat();
        }

        function saveApiKey() {
            const input = document.getElementById('apiKeyInput').value.trim();
            if (input) {
                localStorage.setItem(API_KEY_STORAGE, input);
                apiKey = input;
                document.getElementById('status').textContent = '✅ Ключ сохранен';
                setTimeout(() => document.getElementById('status').textContent = '', 2000);
            }
        }

        async function sendMessage() {
            const input = document.getElementById('messageInput');
            const text = input.value.trim();
            if (!text) return;
            if (!apiKey) {
                alert('Введи API-ключ DeepSeek в поле выше');
                return;
            }
            addUserMessage(text);
            input.value = '';
        }

        async function generateDiary() {
            const entries = getEntries().filter(e => e.role === 'user');
            if (entries.length === 0) {
                alert('Сегодня еще нет записей. Добавьте мысли в чат.');
                return;
            }
            if (!apiKey) {
                alert('Введи API-ключ DeepSeek');
                return;
            }
            const rawText = entries.map(e => e.content).join('\n');
            const todayStr = new Date().toLocaleDateString('ru-RU');

            const container = document.getElementById('chatContainer');
            const loadingDiv = document.createElement('div');
            loadingDiv.className = 'message assistant';
            loadingDiv.textContent = '⏳ Обрабатываю записи...';
            container.appendChild(loadingDiv);
            container.scrollTop = container.scrollHeight;

            try {
                const response = await fetch('https://api.deepseek.com/v1/chat/completions', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${apiKey}`
                    },
                    body: JSON.stringify({
                        model: 'deepseek-chat',
                        messages: [
                            {
                                role: 'system',
                                content: `Ты — редактор дневника. Преврати сырой поток мыслей пользователя в связный рассказ от первого лица за день ${todayStr}.
Требования:
1. Придумай заголовок дня (краткий, ёмкий).
2. Отредактируй грамматику, убери слова-паразиты, сделай плавные переходы.
3. Важные идеи, инсайты, неожиданные мысли выдели **жирным текстом** (Markdown).
4. В конце добавь раздел «💡 Идеи и выводы дня» с маркированным списком (3-5 пунктов).
5. Используй абзацы, чтобы текст читался легко.
Отвечай только готовым текстом на русском языке.`
                            },
                            { role: 'user', content: rawText }
                        ],
                        temperature: 0.7,
                        max_tokens: 2048
                    })
                });

                const data = await response.json();
                container.removeChild(loadingDiv);
                if (data.choices && data.choices[0]) {
                    const diaryText = data.choices[0].message.content;
                    addAssistantMessage(diaryText);
                } else {
                    addAssistantMessage('❌ Ошибка: ' + (data.error?.message || 'Неизвестная ошибка'));
                }
            } catch (e) {
                container.removeChild(loadingDiv);
                addAssistantMessage('❌ Ошибка соединения: ' + e.message);
            }
        }

        function clearDay() {
            if (confirm('Удалить все записи за сегодня?')) {
                const all = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
                delete all[getTodayKey()];
                localStorage.setItem(STORAGE_KEY, JSON.stringify(all));
                renderChat();
            }
        }

        renderChat();
    </script>
</body>
</html>
