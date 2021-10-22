---
#title: "Servidor Nginx"
date: 2021-10-22T19:09:20+02:00
draft: false
tags : ["Servicios"]
---

# *Práctica: Servidor Web Nginx*

**Tarea 1 (1 punto)(Obligatorio): Crea una máquina del cloud con una red pública. Añade la clave pública del profesor a la máquina. Instala el servidor web nginx en la máquina. Modifica la página index.html que viene por defecto y accede a ella desde un navegador.**
* Entrega la ip flotante de la máquina para que el profesor pueda acceder a ella.
* Entrega una captura de pantalla accediendo a ella

 Ya hemos creado la maquina, hemos añadido la ip publica del profesor a la maquina. Ahora vamos a instalar el servidor web nginx en nuestra maquina del cloud:

>sudo apt-get update

>sudo apt-get install nginx

 Abrimos el puerto 22 para conectarnos por ssh y el puerto 80 para poder ver las webs.

 Mi ip flotante es: 172.22.200.192

 Probamos el acceso a la maquina:
!["IMAGEN1"](/dhcp/IMAGEN-1.png)

 Vamos a modificar la pagina que viene por defecto, que esta en:

>root@servidorwebnginx:/var/www/html# nano index.html

 En nuestra maquina fisica, en el /etc/hosts hemos añadido la siguiente linea:
172.22.201.25 localhost

 Y ahora, vamos a visualizarla en el navegador:
!["IMAGEN2"](/images/IMAGEN-2.png)

**Virtual Hosting**

 Queremos que nuestro servidor web ofrezca dos sitios web, teniendo en cuenta lo siguiente:

* Cada sitio web tendrá nombres distintos.
* Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

 Los dos sitios web tendrán las siguientes características:

* El nombre de dominio del primero será <code>www.iesgn.org</code>, su directorio base será /srv/www/iesgn y contendrá una página llamada index.html, donde sólo se verá una bienvenida a la página del Instituto Gonzalo Nazareno.

* En el segundo sitio vamos a crear una página donde se pondrán noticias por parte de los departamento, el nombre de este sitio será <code>departamentos.iesgn.org</code>, y su directorio base será /srv/www/departamentos. En este sitio sólo tendremos una página inicial index.html, dando la bienvenida a la página de los departamentos del instituto.

 Vamos a configurar todo lo necesario para que nos muestre las dos paginas. Empezamos creando los VirtualHosting:

>root@servidorwebnginx:/etc/nginx/sites-available# cp default iesgn
>root@servidorwebnginx:/etc/nginx/sites-available# ls
>default  iesgn
>root@servidorwebnginx:/etc/nginx/sites-available# cp default departamentos
>root@servidorwebnginx:/etc/nginx/sites-available# ls
>default  departamentos	iesgn

 Modificamos el fichero departamentos:

>server {
>        listen 80;
>
>       root /srv/www/departamentos;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name departamentos.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>        }
>}


 Modificamos el fichero iesgn:

>server {
>        listen 80;
>
>        root /srv/www/iesgn;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name www.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>        }
>}

 Creamos los enlaces simbólicos para la carpeta 'sites-enabled', para asi activar nuestros virtualhosting que hemos creado anteriormente:

>root@servidorwebnginx:/etc/nginx/sites-available# ln -s /etc/nginx/sites-available/iesgn /etc/nginx/sites-enabled/iesgn
>root@servidorwebnginx:/etc/nginx/sites-available# ln -s /etc/nginx/sites-available/departamentos /etc/nginx/sites-enabled/departamentos

 Comprobamos que están bien configurados:

>root@servidorwebnginx:/etc/nginx/sites-available# nginx -t
>nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
>nginx: configuration file /etc/nginx/nginx.conf test is successful

 Vamos a crear la estructura de directorios correspondientes:

>root@servidorwebnginx:/srv# tree
>.
>└── www
>    ├── departamentos
>    │   └── index.html
>    └── iesgn
>        └── index.html
>
>3 directories, 2 files

**Tarea 2 (2 punto)(Obligatorio): Configura la resolución estática en los clientes y muestra el acceso a cada una de las páginas.**

 Configuramos nuestro fichero /etc/hosts en nuestra maquina anfitriona que sera el cliente en este caso:

>172.22.201.25   www.iesgn.org
>172.22.201.25   departamentos.iesgn.org

Reiniciamos el servicio:

>root@servidorwebnginx:/srv# systemctl restart nginx

 Comprobamos que podemos acceder a ellas desde nuestra maquina fisica:
!["IMAGEN3"](/images/IMAGEN-3.png)
!["IMAGEN4"](/images/IMAGEN-4.png)

**Mapeo de URL**

 Cambia la configuración del sitio web www.iesgn.org para que se comporte de la siguiente forma:

**Tarea 3 (1 punto)(Obligatorio): Cuando se entre a la dirección <code>www.iesgn,org</code> se redireccionará automáticamente a <code>www.iesgn.org/principal</code>, donde se mostrará el mensaje de bienvenida. En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido. Muestra al profesor el funcionamiento.**

 Para redireccionar <code>www.iesgn,org</code> a <code>www.iesgn.org/principal</code> necesitamos editar el fichero de configuración de iesgn:

>server {
>        listen 80;
>
>        root /srv/www/iesgn;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name www.iesgn.org;
>
>        location = / {
>                return 301 $scheme://www.iesgn.org/principal;
>                try_files $uri $uri/ =404;
>        }
>
>        location /principal/ {
>                #No se permite la negociacion de contenido
>                try_files $uri $uri/ index.html;
>                #No se permite ver la lista de ficheros
>                autoindex off;
>                #No se permite el seguimiento de enlaces simbolicos
>                disable_symlinks off;
>
>}

 Creamos el directorio principal y movemos a su interior el fichero index.html:

>root@servidorwebnginx:~# mkdir /srv/www/iesgn/principal
>root@servidorwebnginx:~# mv /srv/www/iesgn/index.html /srv/www/iesgn/principal/
>root@servidorwebnginx:~# cd /srv/
>root@servidorwebnginx:/srv# tree
>.
>└── www
>    ├── departamentos
>    │   └── index.html
>    └── iesgn
>        └── principal
>            └── index.html
>
>4 directories, 2 files

 Reiniciamos el servicio de nuevo:

>root@servidorwebnginx:~# systemctl restart nginx.service

 Comprobamos que nos redicciona:
!["IMAGEN5"](/images/IMAGEN-5.png)

**Tarea 4 (1 punto)(Obligatorio): Si accedes a la página www.iesgn.org/principal/documentos se visualizarán los documentos que hay en /srv/doc. Por lo tanto se permitirá el listado de fichero y el seguimiento de enlaces simbólicos siempre que sean a ficheros o directorios cuyo dueño sea el usuario. Muestra al profesor el funcionamiento.**

 Creamos los directorios /srv/doc y /srv/www/iesgn/principal/documentos:

>root@servidorwebnginx:~# mkdir /srv/doc
>root@servidorwebnginx:~# mkdir /srv/www/iesgn/principal/documentos
>root@servidorwebnginx:~# cd /srv/
>root@servidorwebnginx:/srv# tree
>.
>├── doc
>└── www
>    ├── departamentos
>    │   └── index.html
>    └── iesgn
>        └── principal
>            ├── documentos
>            └── index.html
>
>6 directories, 2 files

 Vamos a crear dentro de /srv/doc dos ficheros, uno sera de root y el otro de un usuario debian, tenemos que asignarle los permisos adecuados:

>root@servidorwebnginx:/srv/doc# ls -l
>total 8
>-rw-r--r-- 1 debian debian 71 Jan 26 16:58 pruebadebian.txt
>-rw-r--r-- 1 root   root   69 Jan 26 16:53 pruebaroot.txt

 Creamos el enlace simbólico del contenido que hay en doc hacia la carpeta documentos que esta en /srv/www/iesgn/principal/documentos:

>root@servidorwebnginx:/srv/doc# ln -svf /srv/doc /srv/www/iesgn/principal/documentos
>'/srv/www/iesgn/principal/documentos/doc' -> '/srv/doc'

 Comprobamos que esta bien:

>root@servidorwebnginx:/srv/doc# nginx -t
>nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
>nginx: configuration file /etc/nginx/nginx.conf test is successful

 El siguiente paso es configurar el fichero iesgn que esta en /etc/nginx/sites-availables:

>server {
>        listen 80;
>
>        root /srv/www/iesgn;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name www.iesgn.org;
>
>        location = / {
>                return 301 $scheme://www.iesgn.org/principal;
>                try_files $uri $uri/ =404;
>        }
>
>        location /principal/ {
>                #No se permite la negociacion de contenido
>                try_files $uri $uri/ index.html;
>                #No se permite ver la lista de ficheros
>                autoindex off;
>                #No se permite el seguimiento de enlaces simbolicos
>                disable_symlinks off;
>        }
>
>        location /principal/documentos {
>                #Alias
>                alias /srv/doc;
>                #Permitir el seguimiento de enlaces simbolicos solo al propietario
>                disable_symlinks if_not_owner;
>                #Permitir listado de ficheros
>                autoindex on;
>        }
>
>}

 Reinciciamos el servicio y comprobamos que podemos meternos en la web www.iesgn.org/principal/documentos:

>root@servidorwebnginx:/etc/nginx/sites-available# systemctl restart nginx.service

!["IMAGEN6"](/images/IMAGEN-6.png)

**Tarea 5 (1 punto): En todo el host virtual se debe redefinir los mensajes de error de objeto no encontrado y no permitido. Para el ello se crearan dos ficheros html dentro del directorio error. Entrega las modificaciones necesarias en la configuración y una comprobación del buen funcionamiento.**

 En nuestro virtualhosting creamos un directorio llamado error, y dentro de el crearemos dos ficheros html:

>root@servidorwebnginx:/srv/www/iesgn# tree
>.
>├── error
>│   ├── 403.html
>│   └── 404.html
>└── principal
>    ├── documentos
>    │   └── doc -> /srv/doc
>    └── index.html
>
>4 directories, 3 files

 Vamos a configurar el fichero iesgn. En /principal/documentos cambiamos el autoindex on por autoindex off para asi denegar el acceso:

>server {
>        listen 80;
>
>        root /srv/www/iesgn;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name www.iesgn.org;
>        error_page 404 = /error/404.html;
>        error_page 403 = /error/403.html;
>
>        location = / {
>                return 301 $scheme://www.iesgn.org/principal;
>                try_files $uri $uri/ =404;
>        }
>
>        location /principal/ {
>                #No se permite la negociacion de contenido
>                try_files $uri $uri/ index.html;
>                #No se permite ver la lista de ficheros
>                autoindex off;
>                #No se permite el seguimiento de enlaces simbolicos
>                disable_symlinks off;
>        }
>
>        location /principal/documentos {
>                #Alias
>                alias /srv/doc;
>                #Permitir el seguimiento de enlaces simbolicos solo al propietario
>                disable_symlinks if_not_owner;
>                #Permitir listado de ficheros
>                autoindex off;
>        }
>}

 Reiniciamos el servicio:

>root@servidorwebnginx:/etc/nginx/sites-available# systemctl restart nginx.service

 Comprobamos que funciona correctamente los mensajes de error:
 ERROR 404
!["IMAGEN7"](/images/IMAGEN-7.png)
 ERROR 403:
!["IMAGEN8"](/images/IMAGEN-8.png)

**Tarea 6 (1 punto)(Obligatorio): Añade al escenario otra máquina conectada por una red interna al servidor. A la URL departamentos.iesgn.org/intranet sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL departamentos.iesgn.org/internet, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.**

 Creamos la maquina cliente, su ip flotante es: 172.22.201.48

 Creamos los ficheros y directorios necesarios para los sitios intranet e internet:

>.
>├── index.html
>├── internet
>│   └── index.html
>└── intranet
>    └── index.html
>
>2 directories, 3 files

 Configuramos el servidor en nuestro virtualhost departamentos. Tanto para intranet como para internet:

 Primero vamos a configurar la intranet, podremos acceder desde el cliente pero no podremos acceder desde la maquina anfitriona:

>server {
>        listen 80;
>
>        root /srv/www/departamentos;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name departamentos.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>        }
>
>        location /intranet {
>                allow 10.0.0.8/24;
>                deny all;
>        }
>}

 En el cliente añadimos lo siguiente en el fichero /etc/hosts:

>10.0.0.3 departamentos.iesgn.org

 Antes de comprobarlo reiniciamos servicio en el servidor:

>root@servidorwebnginx:/etc/nginx/sites-available# systemctl restart nginx.service

En nuestro cliente instalamos lynx para poder ver la web desde el cliente:

>sudo apt-get install lynx

 En la siguiente imagen podemos comprobar que desde el cliente podemos acceder pero desde la maquina anfitriona no podemos:
!["IMAGEN9"](/images/IMAGEN-9.png)

 Ahora vamos a configurar internet, desde el cliente no podremos acceder pero desde la maquina fisica si podremos acceder:

>server {
>        listen 80;
>
>        root /srv/www/departamentos;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name departamentos.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>        }
>
>        location /intranet {
>                allow 10.0.0.8/24;
>                deny all;
>        }
>
>        location /internet {
>                allow 172.23.0.0/16;
>                deny all;
>        }
>
>}

 Reiniciamos servicio en el servidor:

>root@servidorwebnginx:/etc/nginx/sites-available# systemctl restart nginx.service

 En la siguiente imagen podremos comprobar que podemos acceder desde la maquina anfitriona pero desde el cliente no podemos acceder:
!["IMAGEN10"](/images/IMAGEN-10.png)

**Tarea 7 (1 punto): Autentificación básica. Limita el acceso a la URL departamentos.iesgn.org/secreto. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente.**

 Vamos a crear los directorios y ficheros necesarios como hacemos siempre:

>root@servidorwebnginx:/srv/www/departamentos# tree
>.
>├── index.html
>├── internet
>│   └── index.html
>├── intranet
>│   └── index.html
>└── secreto
>    └── index.html
>
>3 directories, 4 files

 Instalamos el paquete de apache2-utils porque lo necesitaremos para crear el fichero .htpasswd:

>sudo apt-get install apache2-utils

 Creamos el fichero .htpasswd en el directorio /srv/:

>root@servidorwebnginx:/srv# htpasswd -cb .htpasswd servidor servidor1
>Adding password for user servidor

 Configuramos nuestro fichero de departamentos y le vamos a añadir la opción de autentificación básica. Apuntamos al fichero /srv/.htpasswd, que hemos configurado previamente:

>server {
>        listen 80;
>
>        root /srv/www/departamentos;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name departamentos.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>        }
>
>        location /intranet {
>                allow 10.0.0.8/24;
>                deny all;
>        }
>
>       location /internet {
>                allow 172.23.0.0/16;
>                deny all;
>        }
>
>        location /secreto {
>                auth_basic "Autentificación";
>                auth_basic_user_file /srv/.htpasswd;
>        }
>
>}

 Reiniciamos servicio en el servidor:

>root@servidorwebnginx:/srv/www/departamentos/secreto# systemctl restart nginx.service

 Ahora vamos a comprobar que desde nuestra maquina fisica al acceder a departamentos.iesgn.org/secreto nos pedira usuario y contraseña para mostrarnos la web:
!["IMAGEN11"](/images/IMAGEN-11.png)
!["IMAGEN12"](/images/IMAGEN-12.png)

 Para ver las cabeceras tenemos que instalar el comando curl:

>sudo apt-get install curl

 Para ver las cabeceras lo hacemos con el comando curl y también podemos con el comando wget.

>root@cliente-servidor-nginx:/home/debian# curl -I departamentos.iesgn.org/secreto
>HTTP/1.1 401 Unauthorized
>Server: nginx/1.14.2
>Date: Tue, 26 Jan 2021 19:23:38 GMT
>Content-Type: text/html
>Content-Length: 195
>Connection: keep-alive
>WWW-Authenticate: Basic realm="Autentificación"

>root@cliente-servidor-nginx:/home/debian# wget -S departamentos.iesgn.org/secreto
>--2021-01-26 19:24:49--  http://departamentos.iesgn.org/secreto
>Resolving departamentos.iesgn.org (departamentos.iesgn.org)... 10.0.0.3
>Connecting to departamentos.iesgn.org (departamentos.iesgn.org)|10.0.0.3|:80... connected.
>HTTP request sent, awaiting response... 
>  HTTP/1.1 401 Unauthorized
>  Server: nginx/1.14.2
>  Date: Tue, 26 Jan 2021 19:24:49 GMT
>  Content-Type: text/html
>  Content-Length: 195
>  Connection: keep-alive
>  WWW-Authenticate: Basic realm="Autentificación"
>
>Username/Password Authentication Failed.

**Tarea 8 (2 punto): Vamos a combinar el control de acceso (tarea 6) y la autentificación (tarea 7), y vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL departamentos.iesgn.org/secreto se hace forma directa desde la intranet, desde la red pública te pide la autentificación. Muestra el resultado al profesor.**

 Para hacer esta parte de la practica tenemos que editar el fichero departamentos y añadirle un par de lineas, el fichero quedaría así:

>server {
>        listen 80;
>
>        root /srv/www/departamentos;
>        index index.html index.htm index.nginx-debian.html;
>
>        server_name departamentos.iesgn.org;
>
>        location / {
>                try_files $uri $uri/ =404;
>       }
>
>        location /intranet {
>                allow 10.0.0.8/24;
>                deny all;
>        }
>
>       location /internet {
>                allow 172.23.0.0/16;
>                deny all;
>        }
>
>        location /secreto {
>                satisfy any;
>                allow 10.0.0.0/24;
>                auth_basic "Autentificación";
>                auth_basic_user_file /srv/.htpasswd;
>        }
>}

 Reiniciamos servicio del servidor nginx:

>root@servidorwebnginx:~# systemctl restart nginx.service

 En las siguientes imagenes podemos comprobar que desde fuera nos pide usuario y contraseña y desde dentro no nos pide nada, nos muestra la web directamente:
!["IMAGEN13"](/images/IMAGEN-13.png)
!["IMAGEN14"](/images/IMAGEN-14.png)

!["IMAGEN15"](/images/IMAGEN-15.png)
