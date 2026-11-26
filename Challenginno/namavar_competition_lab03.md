# Challenge Overview

| Platform & Name | Challenginno / NamAvar-Competition-lab01 |
| --------------- | ---------------------------------------- |
| Date            | 2026-11-25                               |
| Solver          | ph4nt01                                  |
| 🔰 Category     | Network / Web                            |
| ⭐ Difficulty    | Easy                                  |
| 🎯 Points       | 5                                        |

---

# Initial Info

![img](./imgs/lab03.png)

1. After identifying the port and sending a request to it, find the flag value. (5 points)

---

# Initial Analysis

First i ran `nmap -p 8000-8200` to find the target service and port for the flag objective
![imgs](./imgs/13.png)

---

# Solving

After that i found the flag via sending a simple `GET` request via `curl` to the target server
![imgs](./imgs/14.png)

---

# Takeaways

- Always checkout the HTTP headers, there may be something intersting there

---

