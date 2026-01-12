# MSSQL (1433)

---

# **🔍 Enumeración MSSQL**

## **Metasploit:**

- `auxiliary/admin/mssql/mssql_enum`: Enumeración avanzada.
- `auxiliary/admin/mssql/mssql_enum_sql_logins`: Enumera logins.
- `auxiliary/admin/mssql/mssql_enum_domain`: Enumera cuentas de dominio.

## **Nmap:**

- `nmap -Pn -p 1433 --script ms-sql-info <ip>`: Info general (banner).
- `nmap -Pn -p 1433 --script ms-sql-ntlm-info <ip>`: Dumpea info de dominio/hostname.
- `nmap -Pn -p 1433 --script ms-sql-dump-hashes --script-args mssql.username=<u>,mssql.password=<p> <ip>`: Dumpea hashes.

# ⌨️ Acceso MSSQL

## **Metasploit:**

`auxiliary/scanner/mssql/mssql_login`: Fuerza bruta.

## **Nmap:**

`nmap -Pn -p 1433 --script ms-sql-brute --script-args userdb=<u>,passdb=<p> <ip>`: Fuerza bruta.

`nmap -Pn -p 1433 --script ms-sql-empty-password <ip>`: Comprueba login vacío.

# 💥 Explotación MSSQL

## **Metasploit:**

`auxiliary/admin/mssql/mssql_exec`: Ejecuta comandos en el servidor.

## **Nmap:**

- `nmap -Pn -p 1433 --script ms-sql-query --script-args mssql.username=<u>,mssql.password=<p>,ms-sql-query.query=<q> <ip>`: Ejecuta query.
- `nmap -Pn -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=<u>,mssql.password=<p>,ms-sql-xp-cmdshell.cmd="<cmd>" <ip>`: Ejecuta comando de sistema (`xp_cmdshell`).