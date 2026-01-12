# SMTP (25)

# **🔍** Enumeración HTTP

### **Metasploit:**

- `auxiliary/scanner/smtp/smtp_enum`: Enumera usuarios SMTP.

### **Telnet / Netcat:**

- `telnet | nc “ip” “puerto_smtp”` Conectarse al servicio
- `VRFY <user>` o `RCPT <user>`: Si responde `OK` o `252`, el usuario es válido.
- `EXPN <user>`: Lista todos los emails disponibles.

### **SMTP-User-Enum:**

- `smtp-user-enum -M <modo> -U <dicc_usuarios> -t <ip>`