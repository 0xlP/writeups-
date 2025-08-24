# Blog 
La sala "Blog de TryhackMe es una máquina diseñada para practicar habilidades
de pentesting, incluyendo reconocimiento, enumeración, explotación y escalamiento de privilegios. En este 
writeup, documentaré el proceso paso a paso para completar la sala, explicando la herramientas utilizadas, 
los resultados obtenidos y las lecciones aprendidas. 
---
## 1. Fase de reconocimiento 
### Escaneo de Puertos con Nmap
El primer paso en cualquier prueba de penetración es identificar los puertos abiertos, servicios y versiones 
que corren en la maquiina objetivo. Para ello, utilice **Nmap** con el siguiente comando:
nmap -Pn -sS -sV -T4 -vv $IP -oN 
|PORT | STATE | SERVICE | REASON | VERSION|
|-----|-------|---------|-----------|--------|
|22/tcp |  open | ssh | syn-ack  ttl 60 | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)|
|80/tcp | open | http | syn-ack  ttl 60 | Apache httpd 2.4.29 ((Ubuntu))|
|139/tcp | open | netbios-ssn | syn-ack ttl 60 | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)|
|445/tcp | open | netbios-ssn | syn-ack ttl 60 | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)|


