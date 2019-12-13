# Practica Docker
## Iván Pérez Fita
### Introducción
Docker es un proyecto de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores de software, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos.​ Docker utiliza características de aislamiento de recursos del kernel Linux, tales como cgroups y espacios de nombres (namespaces) para permitir que "contenedores" independientes se ejecuten dentro de una sola instancia de Linux, evitando la sobrecarga de iniciar y mantener máquinas virtuales.
## MySQL & Wordpress
### Comandos
```bash
docker network create my_net
```
 Crea la network por la que se contectaran los 2 containers
```bash
docker volume create vol_mysql

docker volume create vol_wordpress
```
Se crean los volumenes donde se guardaran los datos de los contenedores de wordpress y de MySql

```bash
docker run --name mysql_practice \
 -e MYSQL_ROOT_PASSWORD=12345678 \
 -e MYSQL_DATABASE=wordpress \
 -e MYSQL_USER=wordpress \ 
 -e MYSQL_PASSWORD=wordpress \ 
 -v vol_mysql:/var/lib/mysql \ 
 -p 3306:3306 \
 -h172.26.0.2 \
 --net=my_net \ 
 mysql:5.7
```
Este comando ejecuta el contenedor con MySQL se compone de:

- --name: Indica el nombre del contenedor
- -e: Indica las variables de entorno
- -v: Indica el volumen que vamos a utilizar (Previamente creado) y se asocia al path /var/lib/mysql
- -p: Enruta los puertos en este caso el puerto 3306 del contendor con el 3306 de nuestra maquina
- -h: Indica la ip del contendor esto nos sera util para que wordpress se pueda conectar a el
- --net: Indica la network que va a usar (Previamente creada)
- Finalmente indicamos la imagen que se va a utilizar. Se esta usando la version 5.7 de MySQL ya que la version latest daba problemas al conectarse con Wordpress

```bash
docker run --name wordpress_practice \
 -e WORDPRESS_DB_HOST=172.26.0.2:3306 \
 -e WORDPRESS_DB_USER=wordpress \
 -e WORDPRESS_DB_PASSWORD=wordpress \ 
 -e WORDPRESS_DB_NAME=wordpress \ 
 --net=my_net \ 
 -p 80:80 \ 
 -v vol_wordpress:/var/www/html \
 wordpress:latest
```

Este comando ejecuta el contendor de Wordpress de compone de:

- --name: Igual que en anterior indicamos el nombre del contenedor
- -e: Indicamos las variables de entorno. En la variable de entorno WORDPRESS_DB_HOST indicamos la ip que hemos puesto en el contenedor de MySQL con el parametro -h
- -p: Enrutamiento de puertos igual que el anterior pero en este caso del puerto 80
- -v: Indicamos el volumen en este caso el path va a /var/www/html
- Finalmente igual que en anterior indicamos la imagen a utilizar

### Funcionalidad
Con estos comandos ya podemos ejecutar los 2 containers:

<img src="image1.png">

Al acceder a localhost podremos ver la pagina de instalación de wordpress

<img src="image2.png">

Configuramos la cuenta y podremos acceder tanto a la pagina como al panel de administración

<img src="image3.png">
<img src="image4.png">

Finalmente vamos a comprobar que la información se esta guardando correctamente asi que vamos a apagar los 2 contendores.
El de wordpress de puede apagar con ctrl + c
El de MySQl hay que apagarlo con el comando:
```bash 
docker stop mysql_practice
```
Ahora limpiamos el ordenador de los containers antiguos con:

```bash
docker rm mysql_practice
docker rm wordpress_practice
```

Finalmente volvemos a crear los contedores con el comando mostrado anteriormete y al entrar a localhost

<img src="image5.png">

Veremos directamente la pagina que habiamos creado sin pasar por la instalación asi que los datos se estan guardando correctamente en los volumenes

## Apache

### Archivos

Vamos a crear 4 archivos

000-default.conf: Este sera el archivo de configuración de site en apache. Se usa 000-default.conf para ahorrarnos tener que hacer el enlace simbolico

<img src="image6.png">

404.html: Pagina de error de la pagina que vamos a mostrar con apache

<img src="image7.png">

index.html: Index de la pagina de apache

<img src="image8.png">

Finalmente el mas importante Dockerfile:

<img src="image9.png">

### Dockerfile

FROM ubuntu
Indicamos que vamos a usar la imagen de ubuntu

RUN apt-get update
Ejecutamos un update para bajarnos los paquetes correctamente

RUN apt-get install apache2 -y 
Instalamos apache

COPY 000-default.conf /etc/apache2/sites-available/000-default.conf
Copiamos el 000-default.conf a el site de apache

COPY index.html /var/www/html/index.html
Copiamos el index.html para que se muestre al ejecutar el apache

COPY 404.html /var/www/html/404.html
Copiamos el 404.html para mostrarlo en caso de error

CMD apache2ctl -D FOREGROUND
Ejecutamos el apache cuando se cree el contenedor el FOREGROUND se indica ya que si se ejecuta en background el contendor se cerraria

EXPOSE 80
Exponemos el puerto 80

### Contenedor

Antes de nada vamos a crear un archivo donde pasaremos los logs del contendor

```bash
mkdir dockerlog
```
Ejecutamos el 
```bash
docker build .
```

Finalmente realizamos el run

```bash
docker run -v /home/ivan/dockerlog:/var/log/apache2/ -p 80:80 8ed3989330c0
```
- -v: Indicamos el volumen en este caso guardaremos los archivos de /var/log/apache2 en /home/ivan/dockerlog
- -p: Enrutamos el puerto 80
- Finalmente ponemos el id de la imagen que se ha creado

Ahora podremos acceder a locahost para ver index.html

<img src="image10.png">

Podremos ver la pagina de error

<img src="image11.png">

Y si vamos a la carpeta /home/ivan/dockerlog veremos los logs

<img src="image12.png">
