
Used to Scan open ports on an IP. Has the following flags:

- -p -> Limit a range of ports to scan:
	- -p WWWW, ZZZZ -> Scan port WWWW and ZZZZ
	- -p XXXX-YYYY -> Scan beetween XXXX port and YYYY port
	- -p- -> Scan all ports

- -s -> Specify scan techniques:
	- -sS -> TCP SYN Scan
	- -sT -> TCP Connect Scan (use if -sS doesn't work)
	- -sU -> UDP Scan
	- -sO -> IP Protocol Scan (doesn't scan ports, returns used IP Protocols)
	- -sV -> Service/Version Scan (doesn't scan ports, returns info about Service/Version)
	- -sC -> Script Scan (uses Custom Scripts in the scan)

- --min-rate/max-rate NNNN -> Send packets no slower/faster than NNNN seconds

---

An example of nmap command: 

```zsh
nmap -p- --min-rate 1000 -sV -sC WW.XXX.YY.ZZZ
```