<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Elite AI Management System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.4.2/mammoth.browser.min.js"></script>
    <style>
        :root { --gold: #c5a059; --dark: #2d1b0d; --cream: #f9f7f2; }
        * { box-sizing: border-box; }
        body, html { height: 100%; margin: 0; padding: 0; overflow: hidden; background-color: var(--cream); }
        .sidebar { background-color: var(--dark); width: 300px; flex-shrink: 0; height: 100vh; color: white; }
        .main-content { flex-grow: 1; height: 100vh; display: flex; flex-direction: column; background-color: var(--cream); }
        .chat-area { flex-grow: 1; overflow-y: auto; padding: 2rem; display: flex; flex-direction: column; gap: 1rem; }
        .gold-btn { background: var(--gold); color: white; border-radius: 8px; font-weight: bold; transition: 0.3s; }
        .gold-btn:hover { background: #b38f48; transform: translateY(-1px); }
        .message { padding: 1rem 1.5rem; border-radius: 15px; max-width: 80%; line-height: 1.6; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
        .message-bot { background: #f1ede4; border-right: 5px solid var(--dark); align-self: flex-start; }
        .message-user { background: white; border-right: 5px solid var(--gold); align-self: flex-end; }
        .input-panel { background: white; border-top: 4px solid var(--gold); padding: 1.5rem; }
    </style>
</head>
<body class="flex">

    <aside class="sidebar p-6 flex flex-col shadow-2xl">
        <div class="text-center mb-8 border-b border-white/10 pb-6">
            <h1 class="text-3xl font-bold" style="color: var(--gold)">Elite BMS</h1>
            <p class="text-[10px] uppercase tracking-widest text-gray-400">AI Enterprise System</p>
        </div>

        <div class="mb-6">
            <label class="text-xs text-gold/80 block mb-2 font-bold tracking-wider">סקיל פעיל</label>
            <select id="activeSkill" class="w-full bg-white/10 border border-white/20 p-3 rounded-lg text-sm text-white outline-none focus:ring-1 focus:ring-gold">
                <option value="">צ'אט כללי</option>
            </select>
            <button onclick="openSkillModal()" class="w-full mt-4 text-[11px] py-2 border border-dashed border-gold/40 rounded-lg text-gold hover:bg-gold/10 transition-all">
                <i class="fas fa-magic ml-1"></i> צור סקיל מומחה חדש
            </button>
        </div>

        <nav class="flex-1 space-y-2 overflow-y-auto">
            <button onclick="location.reload()" class="w-full text-right p-3 rounded-lg hover:bg-white/5 flex items-center gap-3">
                <i class="fas fa-plus text-gold"></i> שיחה חדשה
            </button>
            <button class="w-full text-right p-3 rounded-lg hover:bg-white/5 flex items-center gap-3">
                <i class="fas fa-folder-open text-gold"></i> ארכיון דוחות
            </button>
        </nav>

        <div class="mt-auto p-4 bg-black/30 rounded-xl border border-white/5">
            <div class="flex justify-between text-[10px] mb-2 font-bold"><span>עלות שימוש:</span><span id="costDisplay">$0.00</span></div>
            <div class="w-full bg-gray-700 h-1.5 rounded-full overflow-hidden">
                <div id="costBar" class="bg-gold h-full w-0 transition-all duration-500"></div>
            </div>
        </div>
    </aside>

    <main class="main-content">
        <header class="h-20 bg-white border-b flex items-center justify-between px-10 shadow-sm">
            <div>
                <h2 class="text-xl font-bold text-gray-800">ממשק עבודה חכם</h2>
                <div id="skillIndicator" class="text-[10px] text-gray-400 font-bold uppercase">Mode: Default</div>
            </div>
            <div class="flex items-center gap-6">
                <input type="password" id="apiKey" class="text-xs border-2 border-gray-100 p-2 rounded-lg w-64 focus:border-gold outline-none" placeholder="מפתח קלוד (sk-ant-...)">
                <div class="w-10 h-10 rounded-full bg-gold/10 flex items-center justify-center text-gold border border-gold/20 shadow-sm"><i class="fas fa-crown"></i></div>
            </div>
        </header>

        <div id="chatBox" class="chat-area">
            <div class="message message-bot">
                <b>שלום יוחנן,</b><br>
                המערכת פועלת כעת במצב מסך מלא. תוכל להעלות קבצים לניתוח מהיר, או להפעיל סקיל שלמד מהקבצים שלך איך לעבוד.
            </div>
        </div>

        <div class="input-panel">
            <div class="max-w-5xl mx-auto flex flex-col gap-3">
                <div id="fileStatus" class="hidden text-xs font-bold text-gold"><i class="fas fa-check-circle ml-1"></i> קובץ מוכן לשליחה</div>
                <div class="flex gap-4 items-end">
                    <input type="file" id="mainFile" class="hidden" onchange="handleMainFile(this)">
                    <button onclick="document.getElementById('mainFile').click()" class="p-4 bg-gray-100 rounded-xl hover:bg-gray-200 text-gray-500 transition-colors shadow-inner">
                        <i class="fas fa-paperclip text-xl"></i>
                    </button>
                    <textarea id="userInput" class="flex-1 p-4 border-2 border-gray-100 rounded-xl focus:border-gold outline-none shadow-sm transition-all resize-none" rows="1" placeholder="כתוב פקודה או שאלה..."></textarea>
                    <button onclick="sendMessage()" class="gold-btn px-12 py-4 shadow-lg text-lg flex items-center gap-2">
                        <span>שלח</span> <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </div>
        </div>
    </main>

    <div id="skillModal" class="hidden fixed inset-0 bg-black/80 flex items-center justify-center backdrop-blur-sm z-[100] p-4">
        <div class="bg-white rounded-2xl p-8 w-[600px] shadow-2xl border-t-8 border-gold max-h-[90vh] overflow-y-auto">
            <h3 class="text-2xl font-bold mb-2">הגדרת סקיל חכם</h3>
            <p class="text-sm text-gray-500 mb-6">קלוד יציית אך ורק לכללים ולידע שתזין כאן.</p>
            
            <label class="block text-sm font-bold mb-2">שם הסקיל</label>
            <input id="sName" class="w-full border-2 border-gray-100 p-3 rounded-xl mb-4 outline-none focus:border-gold" placeholder="למשל: מנתח פרויקטים ארט-קיר">
            
            <label class="block text-sm font-bold mb-2">הנחיות קבועות</label>
            <textarea id="sPrompt" class="w-full border-2 border-gray-100 p-3 rounded-xl mb-4 outline-none focus:border-gold h-32" placeholder="מה התפקיד שלו? איך עליו לענות?"></textarea>
            
            <label class="block text-sm font-bold mb-2">קבצי ידע (לימוד הסקיל)</label>
            <input type="file" id="skillFiles" class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:bg-gold/10 file:text-gold hover:file:bg-gold/20 mb-6" multiple onchange="processSkillFiles(this)">
            <div id="skillLoadStatus" class="text-xs text-green-600 font-bold mb-4"></div>

            <div class="flex justify-end gap-4">
                <button onclick="closeSkillModal()" class="text-gray-400 font-bold">ביטול</button>
                <button onclick="saveSkill()" class="gold-btn px-8 py-2">שמור סקיל</button>
            </div>
        </div>
    </div>

    <script>
        // הגדרת פונקציות גלובליות מיד כדי למנוע ReferenceError
        let skills = JSON.parse(localStorage.getItem('elite_bms_skills') || '[]');
        let currentFileContext = "";
        let skillKnowledge = "";
        let totalCost = 0;

        function openSkillModal() { 
            document.getElementById('skillModal').classList.remove('hidden'); 
        }

        function closeSkillModal() { 
            document.getElementById('skillModal').classList.add('hidden'); 
        }

        function renderSkills() {
            const select = document.getElementById('activeSkill');
            select.innerHTML = '<option value="">צ\'אט כללי</option>';
            skills.forEach((s, i) => {
                const opt = document.createElement('option');
                opt.value = i;
                opt.innerText = s.name;
                select.appendChild(opt);
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
            if(!name || !prompt) return alert("חובה למלא שם והנחיות");
            skills.push({ name, prompt, knowledge: skillKnowledge });
            localStorage.setItem('elite_bms_skills', JSON.stringify(skills));
            renderSkills();
            closeSkillModal();
            skillKnowledge = "";
        }

        async function sendMessage() {
            const input = document.getElementById('userInput');
            const key = document.getElementById('apiKey').value || localStorage.getItem('claude_key');
            const skillIdx = document.getElementById('activeSkill').value;

            if(!input.value.trim() && !currentFileContext) return;
            if(!key) return alert("נא להזין מפתח API");
            localStorage.setItem('claude_key', key);

            let systemPrompt = "You are a professional business manager.";
            if(skillIdx !== "") {
                const s = skills[skillIdx];
                systemPrompt = `STRICT RULES: ${s.prompt}\nKNOWLEDGE: ${s.knowledge}\nFollow these instructions ONLY.`;
                document.getElementById('skillIndicator').innerText = `Mode: ${s.name}`;
            }

            appendMessage('user', input.value);
            const prompt = currentFileContext ? `DATA:\n${currentFileContext}\n\nUSER:${input.value}` : input.value;
            
            input.value = "";
            currentFileContext = "";
            document.getElementById('fileStatus').classList.add('hidden');

            try {
                const res = await fetch("https://cors-anywhere.herokuapp.com/https://api.anthropic.com/v1/messages", {
                    method: 'POST',
                    headers: { 'x-api-key': key, 'anthropic-version': '2023-06-01', 'content-type': 'application/json' },
                    body: JSON.stringify({
                        model: "claude-3-5-sonnet-20240620",
                        max_tokens: 4000,
                        system: systemPrompt,
                        messages: [{ role: "user", content: prompt }]
                    })
                });

                const data = await res.json();
                appendMessage('bot', data.content[0].text);
                updateStats(data.usage);
            } catch (e) {
                appendMessage('bot', "שגיאה. וודא שאישרת גישה בכתובת https://cors-anywhere.herokuapp.com/corsdemo");
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
            currentFileContext = await readFileContent(input.files[0]);
        }

        function appendMessage(role, text) {
            const box = document.getElementById('chatBox');
            const div = document.createElement('div');
            div.className = `message ${role === 'user' ? 'message-user' : 'message-bot'}`;
            
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

        // הפעלה ראשונית
        window.onload = () => {
            renderSkills();
            document.getElementById('apiKey').value = localStorage.getItem('claude_key') || "";
        };
    </script>
</body>
</html>
