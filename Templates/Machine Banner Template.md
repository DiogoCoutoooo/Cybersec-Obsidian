<%*
const colors = { "Easy": "#9acd32", "Medium": "#ff8c00", "Hard": "#ff4500", "Insane": "#ffffff" };
const name = await tp.system.prompt("Nome da Máquina");
const diff = await tp.system.suggester(["Easy", "Medium", "Hard", "Insane"], ["Easy", "Medium", "Hard", "Insane"]);
const status = await tp.system.suggester(["PWNED", "USER PWNED", "SYSTEM PWNED", "ACTIVE"], ["PWNED", "USER PWNED", "SYSTEM PWNED", "ACTIVE"]);
const vector = await tp.system.prompt("Vector Principal");
const os = await tp.system.suggester(["Linux", "Windows"], ["Linux", "Windows"]);
const color = colors[diff];
const imgPath = "Tought Process/!Media/!Logo_" + name + ".png";
%><div style="background: linear-gradient(90deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 2px solid <% color %>; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; gap: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.5);">
<div style="flex-shrink: 0; width: 90px; height: 90px; border-radius: 50%; border: 3px solid <% color %>; overflow: hidden; display: flex; justify-content: center; align-items: center; background: #0d1117;">
<img src="<% imgPath %>" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
</div>
<div style="flex-grow: 1;">
<h1 style="margin: 0; font-size: 28px; color: #ffffff; text-transform: uppercase; letter-spacing: 2px;"><% name %> <span style="font-size: 12px; color: <% color %>; border: 1px solid <% color %>; padding: 2px 8px; border-radius: 4px; vertical-align: middle; margin-left: 10px;"><% status %></span></h1>
<div style="display: flex; gap: 20px; margin-top: 12px; font-size: 13px;">
<span><span style="color: <% color %>;">💻</span> <b>OS:</b> <% os %></span>
<span><span style="color: <% color %>;">🔥</span> <b>Diff:</b> <% diff %></span>
<span><span style="color: <% color %>;">📅</span> <b>Pwned:</b> <% tp.date.now("DD MMM YYYY") %></span>
</div>
</div>
<div style="text-align: right;">
<div style="font-size: 10px; text-transform: uppercase; color: <% color %>; margin-bottom: 5px; letter-spacing: 1px;">Primary Vector</div>
<code style="background: rgba(255, 255, 255, 0.05); padding: 6px 12px; border-radius: 5px; border: 1px solid <% color %>; color: <% color %>; font-size: 11px; font-weight: bold;"><% vector %></code>
</div>
</div>