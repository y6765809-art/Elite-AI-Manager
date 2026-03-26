import express from "express";
import fetch from "node-fetch";
import multer from "multer";
import fs from "fs";

const app = express();
const upload = multer({ dest: "uploads/" });

// 🔐 שים כאן את המפתח שלך (או עדיף ENV)
const API_KEY = process.env.CLAUDE_API_KEY || "PUT-YOUR-KEY-HERE";

app.use(express.json());

// ===== UI =====
app.get("/", (req, res) => {
  res.send(`
<!DOCTYPE html>
<html lang="he">
<head>
<meta charset="UTF-8">
<title>Elite Claude AI</title>
<style>
body{font-family:sans-serif;background:#111;color:#fff;text-align:center;padding:20px;}
input,button{padding:10px;margin:5px;border-radius:8px;border:none;}
#out{white-space:pre-wrap;margin-top:20px;text-align:right;}
</style>
</head>

<body>

<h2>🤖 Elite Claude AI</h2>

<input id="msg" placeholder="כתוב שאלה..." style="width:60%">
<button onclick="send()">שלח</button>

<br><br>

<input type="file" id="file">
<button onclick="upload()">העלה קובץ</button>

<div id="out"></div>

<script>

async function send(){
  const res = await fetch("/chat", {
    method:"POST",
    headers:{"Content-Type":"application/json"},
    body: JSON.stringify({
      message: msg.value,
      system: "אתה עוזר מקצועי וחכם"
    })
  });

  const d = await res.json();
  out.innerText += "\\n\\n🤖 " + (d.content?.[0]?.text || "שגיאה");
}

async function upload(){
  const fd = new FormData();
  fd.append("file", file.files[0]);

  const res = await fetch("/upload", {
    method:"POST",
    body: fd
  });

  const d = await res.json();
  out.innerText += "\\n\\n📄 " + (d.content?.[0]?.text || "שגיאה");
}

</script>

</body>
</html>
`);
});

// ===== CHAT =====
app.post("/chat", async (req, res) => {
  try {
    const { message, system } = req.body;

    const r = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "x-api-key": API_KEY,
        "anthropic-version": "2023-06-01",
        "content-type": "application/json"
      },
      body: JSON.stringify({
        model: "claude-3-5-sonnet-20240620",
        max_tokens: 2000,
        system,
        messages: [
          { role: "user", content: message }
        ]
      })
    });

    res.json(await r.json());

  } catch (e) {
    res.json({ error: e.message });
  }
});

// ===== FILE =====
app.post("/upload", upload.single("file"), async (req, res) => {
  try {
    const content = fs.readFileSync(req.file.path, "utf-8");

    const r = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "x-api-key": API_KEY,
        "anthropic-version": "2023-06-01",
        "content-type": "application/json"
      },
      body: JSON.stringify({
        model: "claude-3-5-sonnet-20240620",
        max_tokens: 2000,
        messages: [
          {
            role: "user",
            content: "נתח את הקובץ:\n" + content
          }
        ]
      })
    });

    res.json(await r.json());

  } catch (e) {
    res.json({ error: e.message });
  }
});

app.listen(3000, () => console.log("🔥 פתוח: http://localhost:3000"));
