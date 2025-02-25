Machine Info




Walkthrough

1. Initial port scan for open port.
```
	- ┌─[✗]─[ghoth@parrot]─[~/HTB/machines/sau]
		└──╼ $sudo nmap -p- --min-rate 10000 10.10.11.224 -oA portScan
		[sudo] password for ghoth: 
		Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 14:42 +0545
		Nmap scan report for 10.10.11.224 (10.10.11.224)
		Host is up (0.27s latency).
		Not shown: 65531 closed tcp ports (reset)
		PORT      STATE    SERVICE
		22/tcp    open     ssh
		80/tcp    filtered http
		8338/tcp  filtered unknown
		55555/tcp open     unknown
		
		Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds

```
---

1. Service scan for the open ports. 
```
		┌─[ghoth@parrot]─[~/HTB/machines/sau]
		└──╼ $nmap -p22,80,8338,55555 -sC -sV -oA serviceScan 10.10.11.224Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 14:43 +0545
		Nmap scan report for 10.10.11.224 (10.10.11.224)
		Host is up (0.27s latency).
		
		PORT      STATE    SERVICE VERSION
		22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
		| ssh-hostkey: 
		|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
		|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
		|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
		80/tcp    filtered http
		8338/tcp  filtered unknown
		55555/tcp open     unknown
		| fingerprint-strings: 
		|   FourOhFourRequest: 
		|     HTTP/1.0 400 Bad Request
		|     Content-Type: text/plain; charset=utf-8
		|     X-Content-Type-Options: nosniff
		|     Date: Mon, 24 Feb 2025 08:58:50 GMT
		|     Content-Length: 75
		|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
		|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
		|     HTTP/1.1 400 Bad Request
		|     Content-Type: text/plain; charset=utf-8
		|     Connection: close
		|     Request
		|   GetRequest: 
		|     HTTP/1.0 302 Found
		|     Content-Type: text/html; charset=utf-8
		|     Location: /web
		|     Date: Mon, 24 Feb 2025 08:58:16 GMT
		|     Content-Length: 27
		|     href="/web">Found</a>.
		|   HTTPOptions: 
		|     HTTP/1.0 200 OK
		|     Allow: GET, OPTIONS
		|     Date: Mon, 24 Feb 2025 08:58:18 GMT
		|_    Content-Length: 0
		1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
		SF-Port55555-TCP:V=7.94SVN%I=7%D=2/24%Time=67BC34A8%P=x86_64-pc-linux-gnu%
		SF:r(GetRequest,A2,"HTTP/1\.0\x20302\x20Found\r\nContent-Type:\x20text/htm
		SF:l;\x20charset=utf-8\r\nLocation:\x20/web\r\nDate:\x20Mon,\x2024\x20Feb\
		SF:x202025\x2008:58:16\x20GMT\r\nContent-Length:\x2027\r\n\r\n<a\x20href=\
		SF:"/web\">Found</a>\.\n\n")%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x2
		SF:0Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection
		SF::\x20close\r\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,60,"HTTP/1\.0\x
		SF:20200\x20OK\r\nAllow:\x20GET,\x20OPTIONS\r\nDate:\x20Mon,\x2024\x20Feb\
		SF:x202025\x2008:58:18\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequ
		SF:est,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/pla
		SF:in;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Reque
		SF:st")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20
		SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\
		SF:x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
		SF:Content-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r
		SF:\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,67,"HTTP/1\.1\x204
		SF:00\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r
		SF:\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TLSSessionReq,6
		SF:7,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x
		SF:20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%
		SF:r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
		SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
		SF:20Request")%r(FourOhFourRequest,EA,"HTTP/1\.0\x20400\x20Bad\x20Request\
		SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nX-Content-Type-Opti
		SF:ons:\x20nosniff\r\nDate:\x20Mon,\x2024\x20Feb\x202025\x2008:58:50\x20GM
		SF:T\r\nContent-Length:\x2075\r\n\r\ninvalid\x20basket\x20name;\x20the\x20
		SF:name\x20does\x20not\x20match\x20pattern:\x20\^\[\\w\\d\\-_\\\.\]{1,250}
		SF:\$\n")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Ty
		SF:pe:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\
		SF:x20Bad\x20Request")%r(LDAPSearchReq,67,"HTTP/1\.1\x20400\x20Bad\x20Requ
		SF:est\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20
		SF:close\r\n\r\n400\x20Bad\x20Request");
		Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
		
		Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
		Nmap done: 1 IP address (1 host up) scanned in 114.76 seconds
```

---


3. 