# 1º - Information Gathering &  Enumeration

# Escaneo Pasivo

### Obtener IP (Búsqueda DNS)

`host <url>`

### Obtener Directorios Ocultos

Busca `robots.txt` y `sitemap_index.xml`. 

Pueden revelar:

- Tecnología (ej. WordPress)
- Directorios ocultos
- Rutas de subida de archivos
- Plugins

### Obtener Tecnologías Web

- **Add-ons (Navegador):** `builtwith` o `wappalyzer`.
- **CLI:** `whatweb <url>` (Puede mostrar plugins de Wordpress).

### Clonar Sitio Web (para ver directorios)

- Instala `HTTrack` y úsalo (tiene UI web) para clonar un sitio.

### Recolectar Información con Whois

- **CLI:** `whois <domain>`
- **Información obtenida:**
    - Nombre del registrante
    - Organización
    - Provincia y país
    - Servidores de nombres (Name servers)
    - Rango de red
    - Nombre de la red (NetName)

### Recolectar Huella con Netcraft

- **Web:** `netcraft.com`
- **Información obtenida:**
    - Tecnologías
    - Info de certificados SSL
    - Vulnerabilidades SSL
    - Infraestructura
    - Historial de hosting
    - Trackers web
    - Info de Whois
    - Bloques de IP
    - Tecnologías de backend

### Recolectar Registros DNS

- **CLI:** `dnsrecon -d <domain_name>`
    - Muestra: `A` (IPv4), `AAAA` (IPv6), `TXT`, `MX` (servidor email), `NS`.
- **Web:** `dnsdumpster.com`
    - Muestra: Servidores DNS, registros MX, subdominios y un **gráfico de la infraestructura**.

### Detección de WAF (Firewall)

- **CLI:** `wafw00f <domain>`
- **Probar todas las tecnologías:** `wafw00f <domain> -a`

### Enumeración de Subdominios (Pasiva)

- **CLI:** `sublist3r -e <buscador_a>,<buscador_b> -d <domain>`
- **Motores:** Baidu, Google, Yahoo, Bing, Ask, Netcraft, Virustotal, ThreatCrowd, PassiveDNS.
- *Nota: Usa VPN, Google tiene rate-limit.*

### Enumerar Subdominios con Google Hacking (Dorks)

- `site:<domain>`: Busca en un dominio específico.
- `site:<domain> inurl:<palabra_clave>`: Busca la palabra clave en la URL (ej. `admin`).
- `site:*.<domain>`: Busca subdominios.
- `site:*.<domain> intitle:<palabra_clave>`: Busca subdominios con una palabra en el título.
- `site:*.<domain> filetype:<tipo>`: Busca archivos (ej. `pdf`, `sql`).
- `intitle:index of`: Muestra listado de directorios (Directory Listing).
- `cache:<domain>`: Muestra la caché de Google.
- `inurl:auth_user_file.txt`: Puede mostrar usuarios/contraseñas.
- `inurl:passwd.txt`: Puede mostrar archivos `passwd` filtrados.
- **Base de datos:** [Exploit-DB Google Hacking Database](https://www.exploit-db.com/google-hacking-database)

### Obtener Emails

- **Herramienta:** `theHarvester` (algunos motores requieren API key).
- **Por dominio:** `theHarvester -d <domain> -b <buscador_a>,<buscador_b>`
- **Por organización:** `theHarvester -d <org_name> -b <buscador_a>,<buscador_b>`

### Bases de Datos de Contraseñas Filtradas

- **Web:** `haveibeenpwned.com` (Muestra si un email ha sido expuesto en una brecha).

---

# Escaneo Activo

## Tipos de Registros DNS

- **A:** Hostname -> IPv4
- **AAAA:** Hostname -> IPv6
- **NS:** Servidor de Nombres
- **MX:** Servidor de Correo
- **CNAME:** Alias de dominio
- **TXT:** Registro de texto
- **HINFO:** Info del Host
- **SOA:** Autoridad del dominio
- **SRV:** Registros de servicio
- **PTR:** IP -> Hostname

## Ataque de Transferencia de Zona DNS

Intenta copiar todos los registros (archivos de zona) de un servidor DNS.

- `dnsrecon -d <domain>`
- `dnsenum <domain>`
- `dig axfr @<nameserver> <domain>`
- `fierce -dns <domain>`

## Descubrimiento de Hosts

- `sudo nmap -sn <rango_de_red>` (Ej: `192.168.1.0/24`)
- `netdiscover -i <interfaz> -r <rango_de_red>`

## Escaneo de Puertos con Nmap

### **Opciones TCP:**

- `nmap <ip>`: Escaneo básico.
- `nmap -Pn <ip>`: No hacer ping (asume que el host está vivo).
- `nmap -p <puerto> <ip>`: Puerto específico.
- `nmap -p <puertoA>,<puertoB> <ip>`: Varios puertos.
- `nmap -p<inicio>-<fin> <ip>`: Rango de puertos.
- `nmap -F <ip>`: Escanea los 100 puertos más comunes (Rápido).
- `nmap <ip> -vv`: Aumenta la verbosidad (da más detalles).
- `nmap -sV <ip>`: Detección de **Versión** de servicios.
- `nmap -O <ip>`: Detección de **Sistema Operativo**.
- `nmap -sC <ip>`: Ejecuta **Scripts** básicos de enumeración.

### **Velocidad escaneo:**

- `T1`, `T2`: Lentos (Evasión de IDS).
- `T3`: Normal (Por defecto).
- `T4`: Agresivo (Rápido).
- `T5`: Insane (Muy rápido, para redes locales).

### **Escaneo UDP:**

- `nmap -sU <ip>`

### **Formatos de Salida:**

- `-oN <archivo>.txt`: Salida normal (texto).
- `-oG <archivo>.grep` : Salida para filtrar (feo de leer)
- `-oX <archivo>.xml`: Salida en XML (útil para importar).
- -oA <

### Uso de Scripts (Nmap Scripting Engine - NSE)

### **Sintaxis principal:**

- `nmap --script=<categoría_o_script> <ip>`
- `nmap --script-help=<categoría_o_script>` (Para ver qué hace un script)

### **Categorías Principales de Scripts:**

- `default`: Scripts seguros, útiles y comunes. Son los que se ejecutan al usar `sC`.
- `vuln`: Comprueba vulnerabilidades muy conocidas (ej. MS17-010, Heartbleed). **¡Muy útil en eJPTv2!**
- `auth`: Intenta encontrar credenciales débiles o accesos anónimos (ej. `ftp-anon`, login anónimo a SMB).
- `discovery`: Intenta descubrir más información sobre la red y los servicios (ej. `smb-enum-shares`, `dns-zone-transfer`).
- `safe`: Scripts que no son intrusivos y no se consideran "ataques".
- `intrusive`: Scripts **NO seguros**. Tienen un alto riesgo de tumbar (crashear) el servicio o el host. **¡Evítalos en el examen a menos que sepas lo que haces!**
- `exploit`: Intenta *activamente* explotar una vulnerabilidad. (Aún más peligroso que `intrusive`).

### **Para ver todos los scripts disponibles:**

- Puedes listar el directorio donde se guardan:
`ls -l /usr/share/nmap/scripts/`

---

# Mapeo de Red

### Pasivo (Sniffing)

- `Wireshark`
- `TCPDump`

### Activo (Obtener IPs Vivas)

- `arp-scan -I <interfaz> -g <red>\<tipo>`
- `nmap -sn <red>\<tipo>`
- `fping -I <interfaz> -g <red>\<tipo> -a 2>/dev/null`

---

---

## Extras

### Buscar scripts de Nmap

`ls -al /usr/share/nmap/scripts/ | grep <protocolo>-*`

### Importar Nmap a Metasploit

1. Crea un workspace: `workspace -a <nombre_workspace>`
2. Escanea con Nmap y guarda en XML: `nmap [OPCIONES] <ip> -oX <archivo_salida.xml>`
3. Importa en Metasploit: `db_import <archivo_salida.xml>`
4. O escanea directamente desde Metasploit: `db_nmap [OPCIONES] <ip>`