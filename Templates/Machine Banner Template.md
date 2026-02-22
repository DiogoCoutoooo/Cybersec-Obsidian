<%*
const colors = { "Easy": "#9acd32", "Medium": "#ff8c00", "Hard": "#ff4500", "Insane": "#ffffff" };
const pwnColors = { "PWNED": "#9acd32", "USER PWNED": "#ff8c00", "NOT PWNED": "#ff4500" };

// Perguntas iniciais
const name = await tp.system.prompt("Nome da Máquina");
const summary = await tp.system.prompt("Resumo da Box (breve descrição)");
const diff = await tp.system.suggester(["Easy", "Medium", "Hard", "Insane"], ["Easy", "Medium", "Hard", "Insane"]);
const status = await tp.system.suggester(["PWNED", "USER PWNED", "NOT PWNED"], ["PWNED", "USER PWNED", "NOT PWNED"]);
const os = await tp.system.suggester(["Linux", "Windows", "MacOS"], ["Linux", "Windows", "MacOS"]);
const pwnDate = await tp.system.prompt("Data do Pwn (DD MMM YYYY)", tp.date.now("DD MMM YYYY"));

// Valores do Radar
const enumVal = await tp.system.prompt("Enum (0-10)", "5");
const realVal = await tp.system.prompt("Real (0-10)", "5");
const cveVal = await tp.system.prompt("CVE (0-10)", "5");
const customVal = await tp.system.prompt("Custom (0-10)", "5");
const ctfVal = await tp.system.prompt("CTF (0-10)", "5");

const color = colors[diff];
const sColor = pwnColors[status];
const imgPath = "Tought Process/!Media/!Logo_" + name + ".png";

const pts = (val, i) => {
    const angle = (Math.PI * 2 * i) / 5 - Math.PI / 2;
    const r = (val / 10) * 48; 
    return `${80 + r * Math.cos(angle)},${85 + r * Math.sin(angle)}`;
};
const polygonPoints = `${pts(enumVal, 0)} ${pts(realVal, 1)} ${pts(cveVal, 2)} ${pts(customVal, 3)} ${pts(ctfVal, 4)}`;
%>---
name: <% name %>
os: <% os %>
difficulty: <% diff %>
status: <% status %>
pwn_date: <% pwnDate %>
summary: "<% summary %>"
matrix_enum: <% enumVal %>
matrix_real: <% realVal %>
matrix_cve: <% cveVal %>
matrix_custom: <% customVal %>
matrix_ctf: <% ctfVal %>
---

<div style="background: linear-gradient(135deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 1px solid <% color %>; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; justify-content: space-between; gap: 5px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); min-height: 190px;">
    
    <div style="display: flex; align-items: center; gap: 25px;">
        <div style="flex-shrink: 0; width: 105px; height: 105px; border-radius: 50%; border: 3px solid <% color %>; overflow: hidden; background: #0d1117; box-shadow: 0 0 20px <% color %>33;">
            <img src="<% imgPath %>" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
        </div>
        <div>
            <h1 style="margin: 0; font-size: 34px; font-weight: 800; color: #ffffff; text-transform: uppercase; letter-spacing: 2px; line-height: 1;"><% name %></h1>
            <div style="margin-top: 10px;"><span style="font-size: 11px; font-weight: bold; color: <% sColor %>; border: 1.5px solid <% sColor %>; padding: 3px 12px; border-radius: 4px; text-transform: uppercase;"><% status %></span></div>
            <div style="display: flex; gap: 15px; margin-top: 18px; font-size: 14px; font-weight: 500; opacity: 0.9;">
                <span><span style="color: <% color %>;"><% os === 'MacOS' ? '🍎' : (os === 'Windows' ? '🪟' : '💻') %></span> <% os %></span>
                <span><span style="color: <% color %>;">🔥</span> <% diff %></span>
                <span><span style="color: <% color %>;">📅</span> <% pwnDate %></span>
            </div>
        </div>
    </div>

    <div style="width: 220px; height: 190px; flex-shrink: 0; display: flex; align-items: center; justify-content: center;">
        <svg width="220" height="190" viewBox="0 0 190 175">
            <circle cx="80" cy="85" r="62" fill="rgba(255,255,255,0.015)" />
            <polygon points="80,25 137,65 115,130 45,130 23,65" fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="0.5" />
            <polygon points="80,40 122,70 106,118 54,118 38,70" fill="none" stroke="rgba(255,255,255,0.07)" stroke-width="0.5" />
            <polygon points="80,55 108,75 97,106 63,106 52,75" fill="none" stroke="rgba(255,255,255,0.07)" stroke-width="0.5" />
            <polygon points="80,70 93,80 88,95 72,95 67,80" fill="none" stroke="rgba(255,255,255,0.07)" stroke-width="0.5" />
            
            <text x="80" y="15" font-size="9" fill="#aaa" text-anchor="middle" font-weight="bold">ENUM</text>
            <text x="142" y="65" font-size="9" fill="#aaa" text-anchor="start" font-weight="bold">REAL</text>
            <text x="115" y="145" font-size="9" fill="#aaa" text-anchor="middle" font-weight="bold">CVE</text>
            <text x="45" y="145" font-size="9" fill="#aaa" text-anchor="middle" font-weight="bold">CUSTOM</text>
            <text x="18" y="65" font-size="9" fill="#aaa" text-anchor="end" font-weight="bold">CTF</text>
            
            <polygon points="<% polygonPoints %>" fill="<% sColor %>33" stroke="<% sColor %>" stroke-width="2.5" stroke-linejoin="round" />
            <circle cx="80" cy="85" r="2.5" fill="<% sColor %>" />
        </svg>
    </div>
</div>

---