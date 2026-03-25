<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Elite AI Enterprise</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.4.2/mammoth.browser.min.js"></script>
    <style>
        :root { --gold: #c5a059; --dark: #2d1b0d; --cream: #f9f7f2; }
        body { background-color: var(--cream); font-family: 'Segoe UI', system-ui; height: 100vh; margin: 0; overflow: hidden; }
        .sidebar { background-color: var(--dark); width: 300px; flex-shrink: 0; display: flex; flex-direction: column; }
        .main-content { flex-grow: 1; display: flex; flex-direction: column; height: 100vh; background: var(--cream); }
        .chat-container { flex-grow: 1; overflow-y: auto; padding: 2rem; scrollbar-width: thin; }
        .gold-btn { background: var(--gold); color: white; transition: 0.3s; border-radius: 8px; }
        .gold-btn:hover { background: #b38f48; transform: translateY(-1px); }
        .message { margin-bottom: 1.5rem; padding: 1.25rem; border-radius: 12px; max-width: 85%; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .message-bot { background: #f1ede4; border-right: 5px solid var(--dark); align-self: flex-start; }
        .message-user { background: white; border-right: 5px solid var(--gold); align-self: flex-end; margin-right: auto; }
        .input-area { background: white; border-top: 4px solid var(--gold); padding: 1.5rem; }
    </style>
</head>
<body class="flex">

    <aside class="sidebar text-white p-6 shadow-2xl">
        <div class="text-center mb-10 border-b border-white/10 pb-6">
            <h1 class="text-3xl font-bold" style="color: var(--gold)">Elite BMS</h1>
            <p class="text-[10px] uppercase tracking-widest text-gray-400">AI Intelligent Enterprise</p>
        </div>

        <div class="mb-8">
            <label class="text-xs text-gold/80 block mb-2 font-bold">סקיל מומחה פעיל:</label>
            <select id="activeSkill" class="w-full bg-white/10 border border-white/20 p-3 rounded-lg text-sm text-white focus:outline-none focus:ring-1 focus:ring-gold">
                <option value="">צ'אט כללי (ללא הגדרות)</option>
            </select>
            <button onclick="openSkillModal()" class="w-full mt-4 text-xs py-2 bg-white/5 border border-dashed border-gold/50 rounded-lg text-gold hover:bg-gold/10 transition-all">
                <i class="fas fa-magic ml-1"></i> צור סקיל מותאם אישית
            </button>
        </div>

        <nav class="space-y-2 flex-1">
            <button onclick="location.reload()" class="w-full text-right p-3 rounded-lg hover:bg-white/5 flex items-center gap-3">
                <i class="fas fa-plus text-gold"></i> שיחה חדשה
            </button>
            <button class="w-full text-right p-3 rounded-lg hover:bg-white/5 flex items-center gap-3">
                <i class="fas fa-history text-gold"></i> היסטוריית דוחות
            </button>
        </nav>

        <div class="mt-auto p-4 bg-black/30 rounded-xl border border-white/5">
            <div class="flex justify-between text-[10px] mb-2 uppercase tracking-tighter"><span>עלות שימוש:</span><span id="costDisplay">$0.00</span></div>
            <div class="w-full bg-gray-700 h-1 rounded-full"><div id="costBar" class="bg-gold h-full w-0"></div></div>
        </div>
    </aside>

    <main class="main-content">
        <header class="h-20 bg-white border-b flex items-center justify-between px-10 shadow-sm">
            <div>
                <h2 id="headerTitle" class="text-xl font-bold text-gray-800">ממשק עבודה חכם</h2>
                <div id="skillIndicator" class="text-[10px] text-gray-400 font-bold">MOD: GENERAL CHAT</div>
            </div>
            <div class="flex items-center gap-6">
                <input type="password" id="apiKey" class="text-xs border-2 border-gray-100 p-2 rounded-lg w-64 focus:border-gold outline-none" placeholder="מפתח קלוד (sk-ant-...)">
                <div class="w-10 h-10 rounded-full bg-gold/10 flex items-center justify-center text-gold border border-gold/20"><i class="fas fa-crown"></i></div>
            </div>
        </header>

        <div id="chatBox" class="chat-container flex flex-col">
            <div class="message message-bot">
                <b>שלום יוחנן,</b><br>
                המערכת מוכנה. תוכל להשתמש בצ'אט חופשי או להפעיל <b>סקיל</b> שלמד מהקבצים שלך איך לבצע משימות ספציפיות.
            </div>
        </div>

        <div class="input-area">
            <div class="max-w-5xl mx-auto">
                <div id="fileInfo" class="hidden mb-2 text-xs font-bold text-gold"><i class="fas fa-check-circle ml-1"></i> קובץ נטען לזיכרון...</div>
                <div class="flex gap-4 items-end">
                    <input type="file" id="uploadFile" class="hidden" onchange="handleMainFile(this)">
                    <button onclick="document.getElementById('uploadFile').click()" class="p-4 bg-gray-100 rounded-xl hover:bg-gray-200 text-gray-500 shadow-sm">
                        <i class="fas fa-paperclip text-xl"></i>
                    </button>
                    <textarea id="userInput" class="flex-1 p-4 border-2 border-gray-50 rounded-xl focus:border-gold outline-none shadow-sm transition-all resize-none" rows="1" placeholder="הקלד פקודה או שאלה..."></textarea>
