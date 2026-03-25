<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Elite AI Enterprise - Management</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.4.2/mammoth.browser.min.js"></script>
    <style>
        :root { --gold: #c5a059; --dark: #2d1b0d; --cream: #f9f7f2; }
        body { background-color: var(--cream); font-family: 'Segoe UI', system-ui; height: 100vh; overflow: hidden; }
        .sidebar { background-color: var(--dark); width: 280px; transition: 0.3s; }
        .chat-area { height: calc(100vh - 180px); overflow-y: auto; scrollbar-width: thin; }
        .gold-btn { background: var(--gold); color: white; border-radius: 8px; font-weight: bold; }
        .gold-btn:hover { background: #b38f48; box-shadow: 0 4px 12px rgba(197, 160, 89, 0.3); }
        .card { background: white; border-radius: 12px; border-top: 5px solid var(--gold); box-shadow: 0 10px 25px -5px rgba(0,0,0,0.1); }
        .message-bot { background: #f1ede4; border-right: 4px solid var(--dark); margin-left: 20%; }
        .message-user { background: white; border-right: 4px solid var(--gold); margin-right: 20%; }
    </style>
</head>
<body class="flex flex-row-reverse">

    <aside class="sidebar flex flex-col p-6 shadow-2xl text-white">
        <div class="mb-10 text-center border-b border-white/10 pb-6">
            <h1 class="text-2xl font-bold" style="color: var(--gold)">Elite BMS</h1>
            <p class="text-[10px] uppercase tracking-widest text-gray-400">AI Intelligent System</p>
        </div>

        <div class="mb-8">
            <label class="text-xs text-gold/70 block mb-2 font-bold">בחירת סקיל פעיל</label>
            <select id="activeSkill" class="w-full bg-white/5 border border-white/10 p-3 rounded-lg text-sm text-white focus:outline-none">
                <option value="">צ'אט אישי (ללא סקיל)</option>
            </select>
            <button onclick="openSkillModal()" class="w-full mt-3 text-xs py-2 border border-dashed border-gold/40 rounded hover:bg-gold/10 text-gold transition-all">
                <i class="fas fa-plus-circle ml-1"></i> יצירת סקיל חכם חדש
            </button>
        </div>

        <nav class="space-y-3 flex-1">
            <button onclick="newChat()" class="w-full text-right p-3 rounded-lg hover:bg-white/5 transition flex items-center gap-3">
                <i class="fas fa-comment-dots text-gold"></i> שיחה חדשה
            </button>
            <button class="w-full text-right p-3 rounded-lg hover:bg-white/5 transition flex items-center gap-3">
                <i class="fas fa-file-invoice-dollar text-gold"></i> ניהול תקציב טוקנים
            </button>
        </nav>

        <div class="p-4 bg-black/20 rounded-xl">
            <div class="flex justify-between text-[10px] mb-2"><span>שימוש נוכחי:</span><span id="costDisplay">$0.00</span></div>
            <div class="w-full bg-gray-700 h-1.5 rounded-full"><div id="costBar" class="bg-gold h-full" style="width: 0%"></div></div>
        </div>
    </aside>

    <main class="flex-1 flex flex-col relative">
        <header class="h-20 bg-white border-b flex items-center justify-between px-10 shadow-sm">
            <div>
                <h2 class="text-xl font-bold">לוח בקרה AI</h2>
                <p id="skillStatus" class="text-xs text-gray-500">מצב עבודה: צ'אט חופשי</p>
            </div>
            <div class="flex gap-4 items-center">
                <input type="password" id="apiKey" class="text-xs border-2 border-gray-100 p-2 rounded-lg w-64 focus:border-gold outline-none" placeholder="הכנס מפתח קלוד (sk-ant-...)">
                <div class="h-10 w-10 rounded-full bg-gold/10 flex items-center justify-center text-gold"><i class="fas fa-user-tie"></i></div>
            </div>
        </header>

        <div id="chatBox" class="chat-area p-10 space-y-6">
            <div class="message-bot p-5 rounded-xl shadow-sm">
                <b>שלום יוחנן,</b><br>המערכת מוגדרת כעת תחת פרוטוקול Elite. תוכל להעלות קבצים לניתוח או להגדיר "סקיל" חדש שילמד מהקבצים שלך איך לעבוד.
            </div>
        </div>

        <div class="p-8 bg-white border-t-4 border-gold">
            <div class="max-w-5xl mx-auto">
                <div id="fileStatus" class="hidden mb-2 text-xs font-bold text-gold animate-pulse"><i class="fas fa-file-import ml-1"></i> קובץ נטען לזיכרון...</div>
                <div class="flex gap-4 items-end">
                    <input type="file" id="mainFile" class="hidden" onchange="handleMainFile(this)">
                    <button onclick="document.getElementById('mainFile').click()" class="p-4 bg-gray-50 rounded-xl hover:bg-gray-100 text-gray-500 shadow-inner">
                        <i class="fas fa-paperclip text-xl"></i>
                    </button>
                    <textarea id="userInput" class="flex-1 p-4 border-2 border-gray-50 rounded-xl focus:border-gold outline-none shadow-sm transition-all resize-none" rows="1" placeholder="שלח הודעה או פקודה..."></textarea>
                    <button onclick="sendMessage()" class="gold-btn px-10 py-4 shadow-lg flex items-center gap-2 text-lg">
                        <span>שלח</span> <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </div>
        </div>
    </main>

    <div id="skillModal" class="hidden fixed inset-0 bg-black/70 flex items-center justify-center backdrop-blur-md p-4 z-50">
        <div class="card p-10 w-[600px] max-h-[90vh] overflow-y-auto">
            <h3 class="text-2xl font-bold mb-2">הגדרת סקיל מומחה</h3>
            <p class="text-sm text-gray-500 mb-6">הסקיל יכלול את הידע מהקבצים והוראות קבועות.</p>
            
            <label class="block text-sm font-bold mb-1">שם הסקיל</label>
            <input id="sName" class="w-full border-2 p-3 rounded-lg mb-4 outline-none focus:border-gold" placeholder="למשל: מנהל חשבונות ארט-קיר">
            
            <label class="block text-sm font-bold mb-1">הנחיות קבועות (מה עליו לעשות?)</label>
            <textarea id="sPrompt" class="w-full border-2 p-3 rounded-lg mb-4 outline-none focus:border-gold h-32" placeholder="כתוב כאן איך על קלוד להתנהג..."></textarea>
            
            <label class="block text-sm font-bold mb-1">קבצי ידע ללימוד (אופציונלי)</label>
            <input type="file" id="skillFiles" class="mb-6 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-gold/10 file:text-gold hover:file:bg-gold/20" multiple onchange="loadSkillFiles(this)">
            <div id="skillFileStatus" class="text-[10px] text-green-600 mb-4 font-bold"></div>

            <div class="flex justify-end gap-4 mt-4">
                <button onclick="closeSkillModal()" class="text-gray-400 font-bold">ביטול</button>
                <button onclick="saveSkill()" class="gold-btn px-8 py-2">שמור והפעל סקיל</button>
            </div>
        </div>
    </div>

    <script>
        let skills = JSON.parse(localStorage.getItem('elite_v2_skills') || '[]');
        let currentKnowledgeBase = "";
        let currentFileContext = "";
        let totalCost = 0;

        // ניהול סקילים
        function openSkillModal() { document.getElementById('skillModal').classList.remove('hidden'); }
        function closeSkillModal() { document.getElementById('skillModal').classList.add('hidden'); }
        
        async function loadSkillFiles(input) {
            const files = input.files;
            let combinedText = "";
            document.getElementById('skillFileStatus').innerText = "מעבד קבצי ידע...";
            
            for (let file of files) {
                const text = await readFile(file);
                combinedText += `\n--- SOURCE: ${file.name} ---\n${text}\n`;
            }
            currentKnowledgeBase = combinedText;
            document.getElementById('
