# Bounty Hacker 
---

## 1. Fase de reconocimiento

 `nmap -Pn -sS -sV -sC -T4 -vv 10.10.149.146`

 ***Resultado:***
| PORT |    STATE | SERVICE |       REASON       |  VERSION |
|------|----------|---------|--------------------|----------|
|20/tcp|closed ftp-data |        reset ttl 63 |
|21/tcp|    open  | ftp  |           syn-ack ttl 63 | vsftpd 3.0.5 | 
|22/tcp|   open  | ssh   |          syn-ack ttl 63  | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0) |
|80/tcp|    open  | http  |          syn-ack ttl 63 | Apache httpd 2.4.41 ((Ubuntu)) |
---
Puerto 80 abierto, significa que una pagina web esta corriendo. 
---
## 2. Fase de enumeracion activa 
Utilizaremos Gobuster para la busqueda de directorios ocultos.

/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/images               (Status: 301) [Size: 313] [--> http://10.10.231.44/images/]
/index.html           (Status: 200) [Size: 969]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.231.44/javascript/]
/server-status        (Status: 403) [Size: 277]
Progress: 4613 / 4613 (100.00%)
