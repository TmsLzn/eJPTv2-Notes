# MySQL (3306)

---

## **🔍 Enumeración MySQL**

### **Comandos MySQL:**

| **Comando** | **Descripción** |
| --- | --- |
| `mysql -h "host" -u "usuario" -p` | Acceso a servicio Mysql |
| `show DATABASES;` | Mostrar bases de datos existentes |
| `use "nombre_db";` | Seleccionar base de datos |
| `show TABLES;` | Mostrar tablas de db actual |
| `describe "nombre_tabla";` | Muestra columnas de tabla |

### **Metasploit:**

| **Módulo** | **Descripción** |
| --- | --- |
| `auxiliary/scanner/mysql/mysql_version` | Obtiene versión. |
| `auxiliary/admin/mysql/mysql_enum` | Enumeración avanzada. |
| `auxiliary/scanner/mysql/mysql_schemadump` | Dumpea bases de datos y tablas. |
| `auxiliary/scanner/mysql/mysql_hashdump` | Dumpea hashes de usuarios. |

### **Nmap:**

- `nmap -Pn -p 3306 --script mysql-info <ip>`: Info general (banner).
- `nmap -Pn -p 3306 --script mysql-users --script-args="mysqluser='u',mysqlpass='p'" <ip>`: Dumpea usuarios.
- `nmap -Pn -p 3306 --script mysql-databases --script-args="mysqluser='u',mysqlpass='p'" <ip>`: Dumpea bases de datos.
- `nmap -Pn -p 3306 --script mysql-variables --script-args="mysqluser='u',mysqlpass='p'" <ip>`: Dumpea variables (puede mostrar directorios).
- `nmap -Pn -p 3306 --script mysql-dump-hashes --script-args="username='u',password='p'" <ip>`: Dumpea hashes.

# ⌨️ Acceso MySQL

## **Hydra:**

```bash
hydra -l <user> -P <dicc_pass> <ip> mysql
```

## **Metasploit:**

```bash
use auxiliary/scanner/mysql/mysql_login
```

## **Nmap:**

```bash
nmap -Pn -p 3306 --script mysql-empty-password <ip>: Comprueba login sin contraseña.
```

# 💥 Explotación MySQL

## **Comandos MySQL:**

`select load_file("archivo");` | Intenta leer un archivo del sistema |

## **Metasploit:**

Lista directorios escribibles. | `auxiliary/scanner/mysql/mysql_writable_dirs` |
Ejecuta una consulta SQL. | `auxiliary/admin/mysql/mysql_sql` |

## **Nmap:**

`nmap -Pn -p 3306 --script mysql-query --script-args="query='<query>',mysqluser='u',mysqlpass='p'" <ip>`: Ejecuta una query.