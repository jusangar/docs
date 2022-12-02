
#### Índice
---
- [Prerrequisitos](#Prerrequisitos)
- [Configuración](#Configuración)
	- [docker-compose.yml](#docker-compose.yml)
	- [pureftpd-mysql.conf](#pureftpd-mysql.conf)
	- [pureftdp.flags](#pureftdp.flags)
- [Instalación](#Instalación)
---

**Pureftpd** es un servidor FTP que soporta el protocolo FXP, la autenticación basada en LDAP / MySQL / PostgreSQL y la encriptación TLS entre otras cosas.

**MariaDB** es una base de datos relacional creada por los desarrolladores originales de MySQL que organiza los datos de forma estructurada.

Documentación:[Pure-FTPd Docker image based on Alpine Linux with MySQL, PostgreSQL and LDAP support](https://github.com/crazy-max/docker-pure-ftpd)


## Prerrequisitos
---

Instalar la última versión de docker compose.

```sh
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

El comando a usar deberá ser `docker compose`, no usar `docker-compose` ya que este comando no usa la versión más reciente.


## Configuración
---

Clonar [este repositorio](https://github.com/jusangar/docker-compose-ftp-server).

```sh
git clone https://github.com/jusangar/docker-compose-ftp-server.git
```

### docker-compose.yml

Cambiar las variables `$[USER]` y `$[PASS]` o crear un archivo _.env_ en el mismo directorio que apunte a esas variables.

Estas variables definen el usuario y la contraseña que gestionará la base de datos.

```env
USER=usuario
PASS=contraseñasegura
```

Si no han sido creadas las networks, comentar `external: true`.

```yml
networks:  
  frontend:  
#    external: true 
  backend:  
#    external: true
```

### pureftpd-mysql.conf

Reemplazar `$[USER]` y `$[PASS]` e introducir el mismo que estableció en _docker-compose.yml_.

```sh
sed 's/$[USER]/usuario/g' pureftpd-mysql.conf
sed 's/$[PASS]/contraseñasegura/g' pureftpd-mysql.conf
```

### pureftdp.flags

En este archivo puede añadir las flags del [manual](https://linux.die.net/man/8/pure-ftpd), la flag por defecto `--tls 2` forzará la conexión TLS. Si no va a usar TLS borre este archivo.

```sh
rm pureftdp.flags
```


## Instalación
---

### Container

Ejecutar el _docker-compose.yml_.

```sh
docker compose up -d
```

Crear un usuario `foo` para auntenticarse por FTP, cambiando `usuario` y `contraseña` con el que puso en el compose.

```sh
docker compose exec db mysql -u usuario -p'contraseña' -e "INSERT INTO users (User,Password,Uid,Gid,Dir) VALUES ('foo',ENCRYPT('test'),'1003','1005','/
```

Verificar que se ha creado el usuario.

```sh
docker compose exec db mysql -u usuario -p'contraseña' -e "SELECT * FROM users;" pureftpd
```

### FTP

Instalar [FTP Rush](https://www.wftpserver.com/ftprush.htm).

![[Pasted image 20221202141053.png]]

Conectarse al servidor FTP con las credenciales del usuario `foo` y el puerto `2100` usar `TLS explícito` si estamos usando un certificado.

## Aclaraciones
---

El certificado TLS deberá ir en la carpeta _data_ con el nombre _pureftpd.pem_.

Hay varias flags forzadas que no se pueden sobrescribir:
- `--bind 0.0.0.0,2100`
- `--ipv4only`
- `--passiveportrange ${PASSIVE_PORT_RANGE}`
- `--noanonymous`
- `--createhomedir`
- `--nochmod`
- `--syslogfacility ftp`

