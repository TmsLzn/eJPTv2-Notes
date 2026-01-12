# SMB (139/445)

---

# Guía Inicial

1. Herramientas Principales
    - **smbclient** y **smbmap**: Interacción con recursos compartidos
    - **rpcclient**: Consultas RPC a servidores SMB
    - **Nmap** + **Scripts NSE**: Enumeración y detección de vulnerabilidades
    - **Metasploit**: Módulos específicos para pruebas
2. Proceso de Auditoría Recomendado
    1. Enumeración inicial con smbclient,smbmap,crackmapexec y Nmap
    2. Análisis de vulnerabilidades con scripts NSE
    3. Enumeración detallada con rpcclient
    4. Post-explotación y montaje de recursos
    5. Búsqueda de información sensible (en otro módulo)

# **🔍** Enumeración SMB

## **Nmap (Scripts):**

- `nmap -Pn -sSVC -p 445 <ip>`: Scan completo.
- `nmap -Pn -p 445 --script smb-protocols <ip>`: (Primero, ver versiones de SMB).
- `nmap -Pn -p 445 --script smb-security-mode <ip>`: Enumera opciones de seguridad.
- `nmap -Pn -p 445 --script smb-enum-sessions <ip>`: Enumera sesiones activas.
- `nmap -Pn -p 445 --script smb-os-discovery <ip>`: Descubre SO, dominio y hostname.
- `nmap -Pn -p 445 --script smb-enum-shares <ip>`: Enumera carpetas compartidas.
- `nmap -Pn -p 445 -script smb-enum-shares,smb-ls --script-args smbusername=<u_con_login>,smbpassword=<p_con_login> <ip>`: Muestra carpetas Y lista su contenido.
- `nmap -Pn -p 445 --script smb-enum-users <ip>`: Enumera usuarios locales.
- `nmap -Pn -p 445 --script smb-enum-groups <ip>`: Enumera grupos locales.
- `nmap -Pn -p 445 --script smb-enum-domains <ip>`: Enumera dominios.
- `nmap -Pn -p 445 --script smb-enum-services <ip>`: Enumera servicios (con login).

## **SMBMap (Clave con creds):**

- `smbmap -H <ip> -L`: Lista unidades (Drives) conectadas.
- `smbmap -H <ip> [-u usuario -p password]`

## **Metasploit:**

- `scanner/smb/smb_version`: Detecta versión.
- `scanner/smb/smb_enumusers`: Lista usuarios.
- `scanner/smb/smb_enumshares`: Lista carpetas compartidas.

## **Enum4linux ( Enumera Users):**

- `enum4linux -a <IP>`
- `enum4linux -o <ip>`: Info del SO.
- `enum4linux -U <ip>`: Lista usuarios.
- `enum4linux -G <ip>`: Lista grupos.
- `enum4linux -S <ip>`: Lista carpetas compartidas.
- `enum4linux -i <ip>`: Info de impresoras.
- `enum4linux -r -u <user> -p <pass>`: Obtiene SIDs (RID cycling).

## **SMBClient (Null Session) :**

- `smbclient -L <ip> -N`: Lista grupos y carpetas (sesión null).
- `smbclient //<ip>/<carpeta> -N`: Entra a una carpeta (sesión null).
- `smbclient //<IP>/<recurso> -U usuario` (Conectar a recurso)

## **RPCClient:**

- `rpcclient -U <username> -N <ip>`: Conecta (sin contraseña).
- `rpcclient> srvinfo`: Muestra info del sistema.
- `rpcclient> enumdomusers`: Muestra usuarios del dominio.
- `rpcclient> enumdomgroups`: Muestra grupos del dominio.
- `rpcclient> lookupnames <user>`: Muestra el SID de un usuario.

## **Nbmlookup:**

- `nmblookup -A <ip>`: Obtiene grupo de trabajo / MAC.

## Script fuzzer shares:

```bash
#!/bin/bash

# Define the target and wordlist location
TARGET="target.ine.local"
WORDLIST="/root/Desktop/wordlists/shares.txt"

# Check if the wordlist file exists
if [ ! -f "$WORDLIST" ]; then
    echo "Wordlist not found:$WORDLIST"
    exit 1
fi

# Loop through each share in the wordlist
while read -r SHARE; do
    echo "Testing share:$SHARE"
    smbclient //$TARGET/$SHARE -N -c "ls" &>/dev/null

    if [ $? -eq 0 ]; then
        echo "[+] Anonymous access allowed for:$SHARE"
    else
        echo "[-] Access denied for:$SHARE"
    fi
done < "$WORDLIST"
```

# 🕹️ Acceso a SMB

### **Conectar (Windows):**

- `net use Z: \<ip>\<recurso>$ <password> /user:<user>`

### **SMBMap:**

- `smbmap -u <user> -p <pass> -d <dir> -H <ip>`: Conexión base.
- `smbmap -H <ip> -r '<unidad>'`: Se conecta a una unidad.
- `smbmap -H <ip> --upload 'archivo_local' 'ruta_remota'`: Sube un archivo.
- `smbmap -H <ip> --download 'archivo_remoto'`: Descarga un archivo.

---

## **Fuerza Bruta**

### **Hydra (Fuerza Bruta):**

- `hydra -l <user> -P <dicc_pass> smb://<ip>`

### **Metasploit:**

- `scanner/smb/smb_login`: Ataque de diccionario para login.

# 💥 Explotación SMB

- **Vuln PipeName:**
    - `use exploit/linux/samba/is_known_pipename`
- **EternalBlue (MS17-010):**
    - *Detección:* `nmap -Pn -p 445 --script smb-vuln-ms17-010 <IP>`
    - *Explotación:* `exploit/windows/smb/ms17_010_eternalblue` o AutoBlue.
- **Ejecución remota (con credenciales):**
    - *Impacket:* `psexec.py DOMINIO/usuario:password@<IP>`
    - *Metasploit:* `exploit/windows/smb/psexec` (usando `SMBPASS` con el hash o password).

# ⌨️ Comándos útiles

```bash
get <archivo_víctima>    # Descargamos archivo
put <archivo_local>      # Subimos archivo

allinfo <archivo_víctima>  # Ver permisos
```