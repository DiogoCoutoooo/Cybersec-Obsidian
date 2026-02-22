
<div style="background: linear-gradient(90deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 2px solid #9acd32; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; gap: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.5);">
<div style="flex-shrink: 0; width: 90px; height: 90px; border-radius: 50%; border: 3px solid #9acd32; overflow: hidden; display: flex; justify-content: center; align-items: center; background: #0d1117;">
<img src="Tought Process/!Media/!Logo_Keeper.png" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
</div>
<div style="flex-grow: 1;">
<h1 style="margin: 0; font-size: 28px; color: #ffffff; text-transform: uppercase; letter-spacing: 2px;">Keeper <span style="font-size: 12px; color: #9acd32; border: 1px solid #9acd32; padding: 2px 8px; border-radius: 4px; vertical-align: middle; margin-left: 10px;">ACTIVE</span></h1>
<div style="display: flex; gap: 20px; margin-top: 12px; font-size: 13px;">
<span><span style="color: #9acd32;">💻</span> <b>OS:</b> Linux</span>
<span><span style="color: #9acd32;">🔥</span> <b>Diff:</b> Easy</span>
<span><span style="color: #9acd32;">📅</span> <b>Pwned:</b> 21 Feb 2026</span>
</div>
</div>
<div style="text-align: right;">
<div style="font-size: 10px; text-transform: uppercase; color: #9acd32; margin-bottom: 5px; letter-spacing: 1px;">Primary Vector</div>
<code style="background: rgba(255, 255, 255, 0.05); padding: 6px 12px; border-radius: 5px; border: 1px solid #9acd32; color: #9acd32; font-size: 11px; font-weight: bold;">a</code>
</div>
</div>

For this Box, I started with the usual nmap (using -sV, -sC, -Pn). The results are the following:

###### (INSERT IMAGE)

Then I came across the main page, with a login form

###### (INSERT IMAGE)

After exploring the page a bit, i ran the gosbuster (using dir mode, looking for php and html pages)

###### (INSERT IMAGE)

# VER 0xdf, RETIRED MACHINE