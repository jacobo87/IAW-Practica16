# IAW - Práctica 16
>IES Celia Viñas (Almería) - Curso 2020/2021   
>Módulo: IAW - Implantación de Aplicaciones Web   
>Ciclo: CFGS Administración de Sistemas Informáticos en Red 

## Práctica 16: «Dockerizar» una aplicación LAMP
Tendremos que crear un archivo [Dockerfile](https://docs.docker.com/engine/reference/builder/) para crear una imagen Docker que contenga una **aplicación web LAMP**. Posteriormente deberá realizar la implantación del sitio web en [Amazon Web Services (AWS)](https://aws.amazon.com/es/) haciendo uso de contenedores Docker y de la herramienta [Docker Compose](https://docs.docker.com/compose/).

### Tareas a realizar
Crearemos un archivo ```dockerfile```: 
- Usaremos una imagen de [Ubuntu](https://ubuntu.com/) que está etiquetada como focal ```FROM ubuntu:focal```.
- Instalamos lo necesario para ejecutar **Apache**, **PHP** y una base de datos **MySQL**.
- :warning: Para realizar la instalación de los paquetes será necesario configurar la variable de entorno ```DEBIAN_FRONTEND``` con el valor ```noninteractive```.
> ```ENV DEBIAN_FRONTEND=noninteractive``` 

- Modificamos el archivo de configuración ```config.php```.
- Modificamos el archivo ```index``` para que sea ```index.php```.
- El puerto de escucha es el **80**.

```
FROM ubuntu:focal

LABEL title="apache-lamp" \
  author="Jacobo Azmani"

ENV DEBIAN_FRONTEND=noninteractive 
ENV TZ=Europe/Madrid

RUN apt-get update \
    && apt-get install -y apache2 \
    && apt-get install -y php \
    && apt-get install -y libapache2-mod-php \
    && apt-get install -y php-mysql

RUN apt install git -y \
    && cd /tmp \
    && git clone https://github.com/josejuansanchez/iaw-practica-lamp \
    && mv /tmp/iaw-practica-lamp/src/* /var/www/html/ \
    && sed -i 's/localhost/mysql/' /var/www/html/config.php \
    && rm /var/www/html/index.html

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"] 
```

### Creación de ```docker-compose.yml```
Utilizaremos la última versión de **MySQL** disponible. Es muy **importante** que para configurar la contraseña de **MySQL** usemos ```--default-authentication-plugin=mysql_native_password```.

Importaremos un script con la base de datos, para ello añadimos un volumen llamado ```./sql:/docker-entrypoint-initdb.d```, donde guardaremos los datos de nuestra aplicación.
```bash
    volumes:
      - mysql_data:/var/lib/mysql
      - ./sql:/docker-entrypoint-initdb.d 
```

Crearemos un archivo **.env** para guardar nuestras variables de entorno.
- El archivo final de ```docker-compose.yml``` quedaría así:

```
version: '3.4'

services:

    mysql:
        image: mysql
        command: --default-authentication-plugin=mysql_native_password
        ports:
            - 3306:3306
        environment:
            - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
            - MYSQL_DATABASE=${MYSQL_DATABASE}
            - MYSQL_USER=${MYSQL_USER}
            - MYSQL_PASSWORD=${MYSQL_PASSWORD}
        volumes:
            - mysql_data:/var/lib/mysql
            - ./sql:/docker-entrypoint-initdb.d
        networks:
            - backend-network
        restart: always
    
    phpmyadmin:
        image: phpmyadmin
        environment:
            - PMA_ARBITRARY=1
        ports:
            - 8080:80
        networks:
            - frontend-network
            - backend-network
        depends_on: 
            - mysql
        restart: always

    apache:
        build: ./apache
        ports:
            - 80:80
        networks:
            - frontend-network
            - backend-network
        depends_on: 
            - mysql
        restart: always

networks:
    frontend-network:
    backend-network:

volumes:
    mysql_data:
```

## REFERENCIAS
- [Jacobo Azmani Github](https://github.com/jacobo87/IAW-Practica-PrestaShop-Docker)
- [José Juan Sanchez](https://josejuansanchez.org/iaw/practica-16/index.html)
- [Pila LAMP](https://github.com/josejuansanchez/iaw-practica-lamp)
- [Dockerfile](https://docs.docker.com/engine/reference/builder/)
- [Docker-compose version](https://docs.docker.com/compose/compose-file/compose-versioning/)
