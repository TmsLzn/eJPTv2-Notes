# FTP ( 21 )

---

# **🔍** Enumeración FTP

## **Nmap:**

- `nmap -Pn -p 21 --script ftp-brute --script-args userdb=<dicc_user> <ip>`: Fuerza bruta.
- `nmap -Pn -p 21 --script ftp-anon <ip>`: Comprueba login anónimo.

## **Hydra:**

- `hydra -l <user> -P <dicc_pass> <ip> ftp`

## **Metasploit:**

- `auxiliary/scanner/ftp/ftp_version`: Obtiene versión.
- `auxiliary/scanner/ftp/ftp_login`: Fuerza bruta.
- `auxiliary/scanner/ftp/anonymous`: Comprueba login anónimo.

# 💥 Explotación FTP

La explotación mas común de FTP será Fuerza Bruta

- **Fuerza Bruta (Hydra):**`bash hydra -L usuarios.txt -P contraseñas.txt -t 4 <IP> ftp`
- **Enumeración/Brute (Nmap):**`bash nmap --script ftp-brute --script-args userdb=users.txt,passdb=pass.txt -p 21 <IP>`

Tambien es importante validar si la **versión de FTP es vulnerable**.

# ⌨️ Comándos útiles

Conexión al objetivo

```jsx
ftp <ip_objetivo>   # Posteriormente introducimos creedenciales
```

Comandos dentro de FTP

```bash
ls     # Listar archivos y directorios
get <nombre_archivo>  # Descargar archivo
put <nombre_archivo>  # Subir archivo
```

Descargar desde bash

```bash
wget -m ftp://user:contraseña@<ip>
```