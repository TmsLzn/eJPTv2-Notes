# SSH (22)

# **🔍 Enumeración SSH**

## **Metasploit:**

- `auxiliary/scanner/ssh/ssh_login`: Fuerza bruta.
- `auxiliary/scanner/ssh/ssh_version`: Obtiene versión (Banner Grabbing).
- `auxiliary/scanner/ssh/ssh_enumusers`: Enumera usuarios.

## **Nmap:**

```bash
ls -lah /usr/share/nmap/scripts | grep "ssh-*" # Ver los scripts de nmap para el servicio ssh
```

- `nmap -Pn -p 22 --script ssh2-enum-algos <ip>`: Enumera algoritmos de cifrado.
- `nmap -Pn -p 22 --script ssh-hostkey <ip>`: Obtiene la clave pública RSA del host.
- `nmap -Pn -p 22 --script ssh-auth-methods --script-args="ssh.user=<user>" <ip>`: Comprueba métodos de autenticación (ej. password, pubkey).
- `nmap -Pn -p 22 --script ssh-brute --script-args userdb=<dicc_user> <ip>`: Fuerza bruta.

## **Hydra:**

- `hydra -l <user> -P <dicc_pass> <ip> ssh`

# 💥 Explotación SSH

Primero deberemos comprobar si la **versión es vulnerable** sino, realizar fuerza bruta:

- Acceder mediante Metasploit: `use auxiliary/scanner/ssh/ssh_login`
- **Fuerza Bruta (Hydra):**`bash hydra -L usuarios.txt -P contraseñas.txt -t 4 <IP> ssh`

# ⌨️ Acceso SSH

## Usuario:Contraseña

```bash
ssh <username>@<ip>   # Proporcionamos posteriormente contraseña
```

## ID_RSA

Si durante una auditoría de seguridad encontramos un archivo `id_rsa`, es posible que no sea necesario conocer la contraseña de un usuario específico. En su lugar, podríamos acceder al servicio utilizando dicha **clave privada**.

### **🔹 Crackeo de Hash SSH con John The Ripper**

```bash
ssh2 johnid_rsa > id_rsa.hash      # Convertimos la clave privada a hash.
john --wordlist= <ruta_diccionario> id_rsa.hash     # Le indicamos el diccionario y el hash.
john --showid_rsa.hash     # Ver la contraseña crackeada
```

> Si la clave privada está protegida, John The Ripper mostrará la contraseña una vez la encuentre. Esta contraseña es necesaria para usar la clave privada al conectarse por SSH.
> 

### **🔹 Cómo conectarnos utilizando la clave `id_rsa`**

Una vez que tengamos acceso al archivo `id_rsa`, debemos seguir estos pasos:

1. **Visualizar el contenido**: Usamos el comando `cat` para ver el contenido del archivo y lo copiamos.
2. **Crear el archivo en nuestra máquina atacante**: En nuestra máquina, creamos un archivo llamado `id_rsa` y pegamos el contenido copiado.
3. **Asignar los permisos correctos**: Para que la clave sea utilizada correctamente, establecemos los permisos adecuados con el siguiente comando:

```bash
chmod 600 id_rsa # Damos permisos para usarla 
```

### **🔹 Iniciar sesión con la clave `id_rsa`**

```bash
sudo ssh -i id_rsa <username>@$<ip_target>  
```