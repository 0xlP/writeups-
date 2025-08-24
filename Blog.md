# Blog 
La sala "Blog de TryhackMe es una máquina diseñada para practicar habilidades
de pentesting, incluyendo reconocimiento, enumeración, explotación y escalamiento de privilegios. En este 
writeup, documentaré el proceso paso a paso para completar la sala, explicando las herramientas utilizadas, 
los resultados obtenidos y las lecciones aprendidas. 
---
## 1. Fase de reconocimiento 
### Escaneo de Puertos con Nmap
El primer paso en cualquier prueba de penetración es identificar los puertos abiertos, servicios y versiones 
que corren en la maquiina objetivo. Para ello, utilice **Nmap** con el siguiente comando:
`nmap -Pn -sS -sV -T4 -vv $IP -oN`
|PORT | STATE | SERVICE | REASON | VERSION|
|-----|-------|---------|-----------|--------|
|22/tcp |  open | ssh | syn-ack  ttl 60 | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)|
|80/tcp | open | http | syn-ack  ttl 60 | Apache httpd 2.4.29 ((Ubuntu))|
|139/tcp | open | netbios-ssn | syn-ack ttl 60 | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)|
|445/tcp | open | netbios-ssn | syn-ack ttl 60 | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)|
- -Pn: No realiza comprobacion de host activo, debido a que muchas maquinas filtan los paquetes ICMP, lo que provoca que nmap lo tome como apagado.
- -sS: Realiza un escaneo SYN, también llamado half-open scan, que es más sigiloso y rápido porque no completa el “three-way handshake” de TCP. En lugar de establecer completamente la conexión, solo envía un paquete SYN inicial y analiza la respuesta del servidor.
- -sV: Activa la detección de versiones de los servicios que se encuentran corriendo en los puertos abiertos. Esto permite identificar la versión exacta del software. Y es fundamental para detectar posibles vulnerabilidades asociadas a esa versión específica. Por ejemplo, algunas versiones antiguas de OpenSSH o Apache pueden tener fallos conocidos que pueden ser explotables.
- -T4: Ajusta la velocidad del escaneo, haciéndolo más rápido sin sobrecargar la red o levantar muchas alertas.
- -vv: Activa la verbosidad extra, mostrando detalles de cada paso del escaneo, útil para depuración y análisis.
- -oN <archivo>: Guarda el resultado del escaneo en un archivo de salida en formato normal, permitiendo consultarlo más tarde o usarlo como referencia en el writeup.

