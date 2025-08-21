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
