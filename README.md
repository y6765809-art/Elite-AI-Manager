<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>Elite Claude BMS - Pro Assistant</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.4.2/mammoth.browser.min.js"></script>
    <style>
        :root { --gold: #c5a059; --brown: #2d1b0d; --cream: #f9f7f2; }
        body { background: var(--cream); font-family: system-ui; }
        .sidebar { background: var(--brown); width: 280px; }
        .gold-border { border-top: 3px solid var(--gold); }
        .btn-gold { background: var(--gold); color: white; }
        .chat-box { height: calc(100vh - 250px); overflow-y: auto; }
    </style>
</head>
<body class="flex h-screen">

    <aside class="sidebar text-white p-5 flex flex-col">
        <div class="text-2xl font-bold text-center mb-8" style="color: var(--gold)">Elite BMS <small class="text-xs block text-gray-400">AI Enterprise</small></div>
        
        <div class="mb-4">
            <h4 class="text-sm text-gray-400 mb-2">ניהול סקילים</h4>
            <select id="skillSelect" class="w-full bg-gray-800 p-2 rounded text-sm mb-2" onchange="loadSkill()">
                <option value="">ללא סקיל (צ'אט רגיל)</option>
            </select>
            <button onclick="openSkillModal()" class="w-full text-xs border border-gray-600 p-1 rounded hover:bg-gray-700">+ הוסף סקיל חדש</button>
        </div>

        <nav class="flex-1 space-y-2 mt-4">
            <button class="w-full text-right p-2 hover:bg-gray-700 rounded"><i class="fas fa-plus ml-2"></i>שיחה חדשה</button>
            <button class="w-full text-right p-2 hover:bg-gray-700 rounded"><i class="fas fa-history ml-2"></i>היסטוריה</button>
        </nav>

        <div class="bg-black/30 p-3 rounded text-xs mt-auto">
            <div class="flex justify-between mb-1"><span>שימוש החודש:</span><span id="costVal">$0.00</span></div>
            <div class="w-full bg-gray-700 h-1 rounded-full"><div id="usageBar" class="bg-gold h-1 w-0"></div></div>
        </div>
    </aside>

    <main class="flex-1 flex flex-col">
        <header class="bg-white p-4 shadow-sm flex justify-between items-center px-8">
            <h1 class="font-bold text-xl">ממשק עבודה חכם</h1>
            <input type="password" id="apiKey" class="text-xs border p-1 rounded w-48" placeholder="הכנס API Key כאן">
        </header>

        <div id="chatArea" class="chat-box p-8 space-y-4">
            </div>

        <div class="p-6 bg-white border-t gold-border">
            <div id="fileInfo" class="text-xs text-blue-600 mb-2 hidden">קובץ נטען...</div>
            <div class="flex gap-3 items-end max-w-5xl mx-auto">
                <input type="file" id="fileInput" class="hidden" accept=".docx,.xlsx,.txt" onchange="handleFile(this)">
                <button onclick="document.getElementById('fileInput').click()" class="p-3 bg-gray-100 rounded-full hover:bg-gray-200">
                    <i class="fas fa-paperclip"></i>
                </button>
                <textarea id="userInput" class="flex-1 border p-3 rounded-xl focus:ring-2 focus:ring-yellow-500 outline-none" rows="2" placeholder="שאל אותי משהו או העלה קובץ לניתוח..."></textarea>
                <button onclick="send()" class="btn-gold px-8 py-3 rounded-xl font-bold shadow-lg">שלח</button>
            </div>
        </div>
    </main>

    <div id="skillModal" class="hidden fixed inset-0 bg-black/50 flex items-center justify-center p-4">
        <div class="bg-white p-6 rounded-lg w-96 shadow-2xl">
            <h3 class="font-bold mb-4">הגדרת סקיל חדש</h3>
            <input id="skillName" class="w-full border p-2 mb-2 rounded" placeholder="שם הסקיל (למשל: מנתח אקסל)">
            <textarea id="skillPrompt" class="w-full border p-2 mb-4 rounded" rows="4" placeholder="הוראות המערכת (System Prompt)..."></textarea>
            <div class="flex justify-end gap-2">
                <button onclick="closeSkillModal()" class="text-gray-500">ביטול</button>
                <button onclick="saveSkill()" class="btn-gold px-4 py-1 rounded">שמור</button>
            </div>
        </div>
    </div>

    <script>
        let skills = JSON.parse(localStorage.getItem('my_skills') || '[]');
        let currentFileContent = "";

        // ניהול סקילים
        function openSkillModal() { document.getElementById('skillModal').classList.remove('hidden'); }
        function closeSkillModal() { document.getElementById('skillModal').classList.add('hidden'); }
        
        function saveSkill() {
            const name = document.getElementById('skillName').value;
            const prompt = document.getElementById('skillPrompt').value;
            if(!name || !prompt) return;
            skills.push({name, prompt});
            localStorage.setItem('my_skills', JSON.stringify(skills));
            renderSkills();
            closeSkillModal();
        }

        function renderSkills() {
            const select = document.getElementById('skillSelect');
            select.innerHTML = '<option value="">ללא סקיל (צ\'אט רגיל)</option>';
            skills.forEach((s, idx) => {
                select.innerHTML += `<option value="${idx}">${s.name}</option>`;
            });
        }

        // טיפול בקבצים
        async function handleFile(input) {
            const file = input.files[0];
            const info = document.getElementById('fileInfo');
            info.innerText = `טוען קובץ: ${file.name}...`;
            info.classList.remove('hidden');

            const reader = new FileReader();
            if (file.name.endsWith('.xlsx')) {
                reader.onload = (e) => {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    currentFileContent = JSON.stringify(XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]]));
                    info.innerText = `קובץ אקסל נטען בהצלחה.`;
                };
                reader.readAsArrayBuffer(file);
            } else if (file.name.endsWith('.docx')) {
                reader.onload = (e) => {
                    mammoth.extractRawText({arrayBuffer: e.target.result}).then(res => {
                        currentFileContent = res.value;
                        info.innerText = `קובץ וורד נטען בהצלחה.`;
                    });
                };
                reader.readAsArrayBuffer(file);
            } else {
                reader.onload = (e) => { currentFileContent = e.target.result; info.innerText = `קובץ טקסט נטען.`; };
                reader.readAsText(file);
            }
        }

        // שליחה לקלוד עם מעקף CORS
        async function send() {
            const text = document.getElementById('userInput').value;
            const key = document.getElementById('apiKey').value || localStorage.getItem('claude_key');
            const skillIdx = document.getElementById('skillSelect').value;
            const systemPrompt = skillIdx !== "" ? skills[skillIdx].prompt : "";

            if(!text || !key) return alert("הזן הודעה ומפתח API");
            localStorage.setItem('claude_key', key);

            appendMsg('user', text);
            document.getElementById('userInput').value = "";

            const fullPrompt = currentFileContent ? `תוכן הקובץ המצורף:\n${currentFileContent}\n\nשאלה:\n${text}` : text;

            try {
                // שימוש ב-Proxy כדי לעקוף חסימה (חשוב!)
                const proxy = "https://cors-anywhere.herokuapp.com/"; 
                const url = "https://api.anthropic.com/v1/messages";

                const response = await fetch(proxy + url, {
                    method: 'POST',
                    headers: {
                        'x-api-key': key,
                        'anthropic-version': '2023-06-01',
                        'content-type': 'application/json'
                    },
                    body: JSON.stringify({
                        model: "claude-3-5-sonnet-20240620",
                        max_tokens: 4096,
                        system: systemPrompt,
                        messages: [{role: "user", content: fullPrompt}]
                    })
                });

                const data = await response.json();
                appendMsg('bot', data.content[0].text);
                currentFileContent = ""; // איפוס קובץ
                document.getElementById('fileInfo').classList.add('hidden');
            } catch (e) {
                appendMsg('bot', "שגיאה: וודא שהפעלת גישה ל-CORS Proxy.");
            }
        }

        function appendMsg(role, text) {
            const div = document.createElement('div');
            div.className = `p-4 rounded-lg max-w-[80%] ${role === 'user' ? 'bg-white mr-auto border-l-4 border-gold' : 'bg-gray-100 ml-auto border-r-4 border-brown'}`;
            div.innerText = text;
            document.getElementById('chatArea').appendChild(div);
            document.getElementById('chatArea').scrollTop = document.getElementById('chatArea').scrollHeight;
        }

        window.onload = () => {
            renderSkills();
            document.getElementById('apiKey').value = localStorage.getItem('claude_key') || "";
        };
    </script>
</body>
</html>
