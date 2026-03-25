<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>Elite AI - מערכת ניהול חכמה</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.4.2/mammoth.browser.min.js"></script>
    <style>
        :root { --gold: #c5a059; --dark: #2d1b0d; --cream: #f9f7f2; }
        html, body { height: 100vh; width: 100vw; margin: 0; padding: 0; overflow: hidden; background: var(--cream); }
        .sidebar { width: 300px; background: var(--dark); height: 100vh; position: fixed; right: 0; top: 0; color: white; z-index: 50; }
        .main-wrapper { margin-right: 300px; height: 100vh; display: flex; flex-direction: column; }
        .header-api { background: white; border-bottom: 2px solid var(--gold); height: 70px; display: flex; align-items: center; justify-content: space-between; padding: 0 2rem; }
        .chat-scroll { flex: 1; overflow-y: auto; padding: 2rem; display: flex; flex-direction: column; gap: 1.5rem; }
        .input-box { background: white; border-top: 3px solid var(--gold); padding: 1.5rem; }
        .msg { padding: 1rem; border-radius: 12px; max-width: 80%; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
        .msg-bot { background: #f1ede4; border-right: 5px solid var(--dark); align-self: flex-start; }
        .msg-user { background: white; border-right: 5px solid var(--gold); align-self: flex-end; }
        .gold-btn { background: var(--gold); color: white; padding: 0.75rem 2rem; border-radius: 8px; font-weight: bold; }
    </style>
</head>
<body>

    <aside class="sidebar p-6 shadow-2xl">
        <div class="text-center mb-10 border-b border-white/10 pb-6">
            <h1 class="text-3xl font-bold" style="color: var(--gold)">Elite BMS</h1>
            <p class="text-[10px] uppercase tracking-widest text-gray-400">Enterprise AI</p>
        </div>

        <div class="mb-8">
            <label class="text-xs text-gold/80 block mb-2 font-bold">סקיל מומחה פעיל:</label>
            <select id="activeSkill" class="w-full bg-white/10 border border-white/20 p-3 rounded-lg text-sm text-white">
                <option value="">צ'אט כללי</option>
            </select>
            <button onclick="openSkillModal()" class="w-full mt-4 text-[11px] py-2 border border-dashed border-gold/40 rounded-lg text-gold hover:bg-gold/10">
                <i class="fas fa-plus-circle ml-1"></i> צור סקיל מומחה
            </button>
        </div>

        <nav class="space-y-3 flex-1">
            <button onclick="location.reload()" class="w-full text-right p-3 rounded-lg hover:bg-white/5 flex items-center gap-3">
                <i class="fas fa-redo text-gold"></i> אתחול שיחה
            </button>
        </nav>

        <div class="p-4 bg-black/30 rounded-xl border border-white/5">
            <div class="flex justify-between text-[10px] mb-2 font-bold uppercase"><span>צריכת תקציב:</span><span id="costDisplay">$0.00</span></div>
            <div class="w-full bg-gray-700 h-1.5 rounded-full overflow-hidden">
                <div id="costBar" class="bg-gold h-full w-0 transition-all duration-500"></div>
            </div>
        </div>
    </aside>

    <div class="main-wrapper">
        <header class="header-api shadow-sm">
            <div>
                <h2 class="font-bold text-gray-800">לוח בקרה חכם</h2>
                <p id="skillNameDisplay" class="text-[10px] text-gray-400 font-bold">מצב: צ'אט רגיל</p>
            </div>
            <div class="flex items-center gap-4 bg-gray-50 p-2 rounded-xl border border-gray-200">
                <span class="text-xs font-bold text-gray-500 mr-2">מפתח API:</span>
                <input type="password" id="apiKey" class="text-sm bg-transparent outline-none w-64" placeholder="הכנס מפתח sk-ant-...">
                <i class="fas fa-key text-gold"></i>
            </div>
        </header>

        <div id="chatBox" class="chat-scroll">
            <div class="msg msg-bot">
                <b>שלום יוחנן,</b><br>
                המערכת מוכנה כעת במסך מלא. הזן את מפתח ה-API למעלה והתחל לעבוד. תוכל להעלות קבצים לניתוח מהיר או להשתמש בסקילים שלך.
            </div>
        </div>

        <div class="input-box">
            <div class="max-w-5xl mx-auto flex flex-col gap-3">
                <div id="fileStatus" class="hidden text-xs font-bold text-gold"><i class="fas fa-paperclip ml-1"></i> קובץ נטען לזיכרון</div>
                <div class="flex gap-4 items-end">
                    <input type="file" id="mainFile" class="hidden" onchange="handleMainFile(this)">
                    <button onclick="document.getElementById('mainFile').click()" class="p-4 bg-gray-100 rounded-xl hover:bg-gray-200 text-gray-400 shadow-inner">
                        <i class="fas fa-upload text-xl"></i>
                    </button>
                    <textarea id="userInput" class="flex-1 p-4 border-2 border-gray-100 rounded-xl focus:border-gold outline-none shadow-sm transition-all resize-none" rows="1" placeholder="כתוב שאלה או פקודה..."></textarea>
                    <button onclick="sendMessage()" class="gold-btn shadow-lg flex items-center gap-2">
                        <span>שלח</span> <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </div>
        </div>
    </div>

    <div id="skillModal" class="hidden fixed inset-0 bg-black/80 flex items-center justify-center backdrop-blur-sm z-[200] p-4">
        <div class="bg-white rounded-2xl p-10 w-[600px] shadow-2xl border-t-8 border-gold max-h-[90vh] overflow-y-auto">
            <h3 class="text-2xl font-bold mb-2">הגדרת סקיל מומחה</h3>
            <p class="text-sm text-gray-500 mb-8">המערכת תציית אך ורק לכללים שתגדיר כאן.</p>
            
            <label class="block text-sm font-bold mb-1">שם הסקיל (למשל: מנתח אקסל)</label>
            <input id="sName" class="w-full border-2 p-3 rounded-lg mb-4 outline-none focus:border-gold" placeholder="תן שם לסקיל...">
            
            <label class="block text-sm font-bold mb-1">הוראות יסוד (System Prompt)</label>
            <textarea id="sPrompt" class="w-full border-2 p-3 rounded-lg mb-4 h-32 outline-none focus:border-gold" placeholder="הגדר את תפקיד הסקיל..."></textarea>
            
            <label class="block text-sm font-bold mb-1">העלה קבצי ידע ללימוד</label>
            <input type="file" id="skillFiles" class="block w-full text-xs text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:bg-gold/10 file:text-gold hover:file:bg-gold/20 mb-6" multiple onchange="processSkillFiles(this)">
            <div id="skillLoadStatus" class="text-xs text-green-600 font-bold mb-4"></div>

            <div class="flex justify-end gap-4 mt-4">
                <button onclick="closeSkillModal()" class="text-gray-400 font-bold">ביטול</button>
                <button onclick="saveSkill()" class="gold-btn px-8 py-2">שמור והפעל</button>
            </div>
        </div>
    </div>

    <script>
        // אתחול גלובלי
        let skills = JSON.parse(localStorage.getItem('elite_bms_vfinal_skills') || '[]');
        let skillKnowledge = "";
        let currentFileData = "";
        let totalCost = 0;

        function openSkillModal() { document.getElementById('skillModal').classList.remove('hidden'); }
        function closeSkillModal() { document.getElementById('skillModal').classList.add('hidden'); }

        function renderSkills() {
            const sel = document.getElementById('activeSkill');
            sel.innerHTML = '<option value="">צ\'אט כללי</option>';
            skills.forEach((s, i) => {
                sel.innerHTML += `<option value="${i}">${s.name}</option>`;
            });
        }

        async function processSkillFiles(input) {
            document.getElementById('skillLoadStatus').innerText = "מעבד קבצי ידע...";
            let text = "";
            for (let f of input.files) {
                text += `\n[קובץ: ${f.name}]\n` + await readFileContent(f);
            }
            skillKnowledge = text;
            document.getElementById('skillLoadStatus').innerText = "הידע נטמע בהצלחה!";
        }

        function saveSkill() {
            const name = document.getElementById('sName').value;
            const prompt = document.getElementById('sPrompt').value;
            if(!name || !prompt) return alert("מלא שם והנחיות");
            skills.push({ name, prompt, knowledge: skillKnowledge });
            localStorage.setItem('elite_bms_vfinal_skills', JSON.stringify(skills));
            renderSkills();
            closeSkillModal();
            skillKnowledge = "";
        }

        async function sendMessage() {
            const input = document.getElementById('userInput');
            const key = document.getElementById('apiKey').value || localStorage.getItem('claude_key');
            const skillIdx = document.getElementById('activeSkill').value;

            if(!input.value.trim() && !currentFileData) return;
            if(!key) return alert("הזן מפתח API למעלה");
            localStorage.setItem('claude_key', key);

            let systemMsg = "You are a professional business system.";
            if(skillIdx !== "") {
                const s = skills[skillIdx];
                systemMsg = `STRICT RULE: ${s.prompt}\nKNOWLEDGE: ${s.knowledge}\nFOLLOW ONLY THESE RULES.`;
                document.getElementById('skillNameDisplay').innerText = `מצב: ${s.name}`;
            }

            appendMsg('user', input.value);
            const userContent = currentFileData ? `DATA:\n${currentFileData}\n\nUSER:${input.value}` : input.value;
            
            input.value = "";
            currentFileData = "";
            document.getElementById('fileStatus').classList.add('hidden');

            try {
                const res = await fetch("https://cors-anywhere.herokuapp.com/https://api.anthropic.com/v1/messages", {
                    method: 'POST',
                    headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'content-type': 'application/json' },
                    body: JSON.stringify({
                        model: "claude-3-5-sonnet-20240620",
                        max_tokens: 4000,
                        system: systemMsg,
                        messages: [{ role: "user", content: userContent }]
                    })
                });

                const data = await res.json();
                appendMsg('bot', data.content[0].text);
                updateStats(data.usage);
            } catch (e) {
                appendMsg('bot', "שגיאה בחיבור. בדוק שהפעלת גישה ב-https://cors-anywhere.herokuapp.com/corsdemo");
            }
        }

        function readFileContent(file) {
            return new Promise((res) => {
                const reader = new FileReader();
                if(file.name.endsWith('.xlsx')) {
                    reader.onload = (e) => {
                        const wb = XLSX.read(e.target.result, {type:'binary'});
                        res(JSON.stringify(XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]])));
                    };
                    reader.readAsBinaryString(file);
                } else if(file.name.endsWith('.docx')) {
                    reader.onload = (e) => mammoth.extractRawText({arrayBuffer: e.target.result}).then(r => res(r.value));
                    reader.readAsArrayBuffer(file);
                } else {
                    reader.onload = (e) => res(e.target.result);
                    reader.readAsText(file);
                }
            });
        }

        async function handleMainFile(input) {
            document.getElementById('fileStatus').classList.remove('hidden');
            currentFileData = await readFileContent(input.files[0]);
        }

        function appendMsg(role, text) {
            const box = document.getElementById('chatBox');
            const div = document.createElement('div');
            div.className = `msg ${role === 'user' ? 'msg-user' : 'msg-bot'}`;
            
            if(text.includes('|') && text.includes('--')) {
                div.innerHTML = text.replace(/\n/g, '<br>') + `<br><button onclick="downloadExcel(this)" class="mt-4 text-[10px] bg-green-700 text-white px-3 py-1 rounded">ייצא כאקסל <i class="fas fa-file-excel"></i></button>`;
            } else {
                div.innerText = text;
            }
            box.appendChild(div);
            box.scrollTo({ top: box.scrollHeight, behavior: 'smooth' });
        }

        function updateStats(usage) {
            const cost = (usage.input_tokens * 0.000003) + (usage.output_tokens * 0.000015);
            totalCost += cost;
            document.getElementById('costDisplay').innerText = `$${totalCost.toFixed(4)}`;
            document.getElementById('costBar').style.width = Math.min(100, totalCost * 100) + "%";
        }

        function downloadExcel(btn) {
            const text = btn.parentElement.innerText;
            const blob = new Blob([text], {type: "application/vnd.ms-excel"});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url; a.download = "Elite_BMS_Report.xls"; a.click();
        }

        window.onload = () => {
            renderSkills();
            document.getElementById('apiKey').value = localStorage.getItem('claude_key') || "";
        };
    </script>
</body>
</html>
