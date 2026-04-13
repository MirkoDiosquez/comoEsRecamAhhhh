# comoEsRecamAhhhh

A — Servidor en puerto distinto
Crear un sitio web simple (HTML)
Hacer que funcione en un puerto raro, por ejemplo:
http://localhost:8080

👉 Básicamente: que el servidor no use el puerto 80 clásico.



-Primer paso : Abrimos la "máquina virtual" con el  
 docker run -ri -p 8080:80 ubuntu:latest bash

 Segundo paso : Instalar Nginx.
 apt update
 apt install nginx

 Tercer paso : Corroborar que este corriendo el nginx.
 
 service nginx status
 
 si no corre, correrlo:

 service nginx start.

 Una vez el servidor corre, Tenemos que acceder a la ip para poder ver el servidor web corriendo. Accedemos con el dominio de:

 http://localhost:8080.


 Veremos la página predeterminada de Nginx. Para modificarla, accedemos al archivo HTML con nano:

 nano /var/www/html/index.nginx-debian.html

 Ahí podemos editar el contenido de la página a gusto. Una vez realizados los cambios, guardamos con Ctrl+O y salimos con Ctrl+X.



 B — Configuración por path
Servir el sitio estático B en un subdirectorio del servidor principal. El sitio debe ser accesible desde: http://localhost/static


Primer paso : Abrimos la "Máquina virutal" con el docker.
docker run -ri -p 8585:85 ubuntu:latest bash

( No es lo más recomendable cambiar el puerto interno )

Esto te deja dentro del contenedor listo para instalar cosas.

Repetimos los mismos paso.

apt update
apt install nginx
apt install nano

service nginx status
service nginx start.


Ahi ya tenemos todo

Ahora hacemos algo distinto.

Segundo paso: Creamos una carpeta en cualquier parte. En nuestro caso, la creamos en /var

mkdir carpetaAuxiliar <- nombre de la carpeta que vamos a utilizar.

Tercer paso : hacemos el archivo html y ponemos cosas adentro para verlo proximamente en la página.

nano /var/carpetaAuxiliar/index.html

Cuarto paso: Una vez hecho el nano, escribimos 3 cosas para ver que nos va a salir y guardamos. Despues de eso escribimos el siguiente comando.

nano /etc/nginx/sites-available/default


ese comando sirve para cambiar la ruta del archivo donde abre la carpeta el cual salta la página predeterminada de Nginx.


nos devolvera algo asi


server {
    listen 80;
listen [::]:80;

location {
    alias /var/www/html;
}

}


Nosotros lo modificamos asi

listen 85;
listen [::]:85;

location /static/ {
    alias /var/carpetaAuxiliar/;
}


Pasos Opcionales :

ls -ld /var/carpetaAuxiliar/
nginx -t

Ambos 2 son para ver si tiene permisos para ver los permisos actuales y si existe un error.


Cuarto paso : Reiniciar nginx 

service nginx restart 

Quinto paso : Corroborar si funciona

http://localhost:8585/static/


C — Configuración por dominio (Virtual Host)
Configurar un Virtual Host para que el sitio C sea accesible mediante un nombre de dominio local. Para esto:


Paso 1 — Instalar NGINX
bashsudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
Verificar que esté corriendo:
bashsudo systemctl status nginx

Paso 2 — Crear la carpeta y el sitio estático
bashsudo mkdir -p /var/www/misitio
sudo nano /var/www/misitio/index.html
Contenido del HTML:
html<html>
  <body>
    <h1>Bienvenido a misitio.com</h1>
  </body>
</html>

Paso 3 — Crear el Virtual Host en NGINX
bashsudo nano /etc/nginx/sites-available/misitio
Contenido del archivo:
nginxserver {
    listen 80;
    server_name misitio.com;

    root /var/www/misitio;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

Paso 4 — Activar el sitio y deshabilitar el default
bashsudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/misitio /etc/nginx/sites-enabled/
Verificar que la configuración no tenga errores:
bashsudo nginx -t
Recargar NGINX:
bashsudo systemctl reload nginx

Paso 5 — Configurar /etc/hosts
bashsudo nano /etc/hosts
Agregar esta línea:
127.0.0.1 misitio.com

Paso 6 — Verificar que funciona
bashcurl http://misitio.com
Resultado esperado: el HTML con "Bienvenido a misitio.com".