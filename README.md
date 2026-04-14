# comoEsRecamAhhhh

Guía de prácticas con NGINX, Docker y Spring Boot.

---

## A — Servidor en puerto distinto

Crear un sitio web simple (HTML) y hacerlo funcionar en un puerto no estándar, por ejemplo:

**http://localhost:8080**

👉 Objetivo: que el servidor no use el puerto 80 clásico.

### Pasos

1) Abrir la "máquina virtual" con Docker:

```bash
docker run -ri -p 8080:80 ubuntu:latest bash
```

2) Instalar NGINX:

```bash
apt update
apt install nginx
```

3) Verificar si NGINX está corriendo:

```bash
service nginx status
```

Si no corre, iniciarlo:

```bash
service nginx start.
```

4) Acceder a:

**http://localhost:8080**

5) Editar la página por defecto de NGINX:

```bash
nano /var/www/html/index.nginx-debian.html
```

Guardar con `Ctrl+O` y salir con `Ctrl+X`.

---

## B — Configuración por path

Servir el sitio estático B en un subdirectorio del servidor principal.  
Debe ser accesible desde:

**http://localhost/static**

### Pasos

1) Abrir la "máquina virtual" con Docker:

```bash
docker run -ri -p 8585:85 ubuntu:latest bash
```

> ( No es lo más recomendable cambiar el puerto interno )

2) Instalar herramientas y verificar NGINX:

```bash
apt update
apt install nginx
apt install nano

service nginx status
service nginx start.
```

3) Crear carpeta auxiliar:

```bash
mkdir carpetaAuxiliar
```

4) Crear el HTML:

```bash
nano /var/carpetaAuxiliar/index.html
```

5) Editar la configuración de NGINX:

```bash
nano /etc/nginx/sites-available/default
```

Configuración de referencia:

```nginx
server {
    listen 80;
listen [::]:80;

location {
    alias /var/www/html;
}

}
```

Configuración aplicada:

```nginx
listen 85;
listen [::]:85;

location /static/ {
    alias /var/carpetaAuxiliar/;
}
```

6) Pasos opcionales de validación:

```bash
ls -ld /var/carpetaAuxiliar/
nginx -t
```

7) Reiniciar NGINX:

```bash
service nginx restart
```

8) Verificar en:

**http://localhost:8585/static/**

---

## C — Configuración por dominio (Virtual Host)

Configurar un Virtual Host para que el sitio C sea accesible por un dominio local.

> ESTE PASO SE HACE DESDE VIRTUAL BOX

### Paso 1 — Instalar NGINX

```bash
bashsudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
```

Verificar estado:

```bash
bashsudo systemctl status nginx
```

### Paso 2 — Crear carpeta y sitio estático

```bash
bashsudo mkdir -p /var/www/misitio
sudo nano /var/www/misitio/index.html
```

Contenido del HTML:

```html
<html>
  <body>
    <h1>Bienvenido a misitio.com</h1>
  </body>
</html>
```

### Paso 3 — Crear Virtual Host en NGINX

```bash
bashsudo nano /etc/nginx/sites-available/misitio
```

Contenido:

```nginx
server {
    listen 80;
    server_name misitio.com;

    root /var/www/misitio;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Paso 4 — Activar sitio y deshabilitar default

```bash
bashsudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/misitio /etc/nginx/sites-enabled/
```

Validar configuración:

```bash
bashsudo nginx -t
```

Recargar NGINX:

```bash
bashsudo systemctl reload nginx
```

### Paso 5 — Configurar `/etc/hosts`

```bash
bashsudo nano /etc/hosts
```

Agregar:

```txt
127.0.0.1 misitio.com
```

### Paso 6 — Verificar

```bash
bashcurl http://misitio.com
```

Resultado esperado: HTML con “Bienvenido a misitio.com”.

---

## D — App Java Spring Boot + Reverse Proxy con NGINX

Crear una app mínima (Hola Mundo) en puerto 8000 y exponerla por NGINX en `/app/`.

Acceso esperado:

**http://<IP-del-servidor>/app/**

### Contexto inicial

Todo se hace dentro de un contenedor Docker.  
No hace falta instalar Java/Maven en la máquina host.

### Paso 1 — Levantar contenedor Ubuntu

```bash
bashdocker run -it --name mi-servidor -p 8888:80 -p 9000:8000 ubuntu:latest bash
```

Qué hace cada parte:

- `-it`: modo interactivo
- `--name mi-servidor`: nombre del contenedor
- `-p 8888:80`: mapea 80 (contenedor) a 8888 (host)
- `-p 9000:8000`: mapea 8000 (contenedor) a 9000 (host)

### Paso 2 — Instalar Java, Maven y NGINX

```bash
bashapt update && apt install -y openjdk-17-jdk maven nginx nano curl
```

Luego verificar:

```bash
bashjava -version && mvn -version && nginx -v
```

### Paso 3 — Estructura del proyecto

```bash
bashmkdir -p ~/holamundo/src/main/java/com/example
mkdir -p ~/holamundo/src/main/resources
cd ~/holamundo
```

### Paso 4 — Crear archivos

#### 4.1 `App.java`

```bash
bashnano src/main/java/com/example/App.java
```

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @GetMapping("/")
    public String holaMundo() {
        return "Hola Mundo desde Spring Boot!";
    }
}
```

#### 4.2 `application.properties`

```bash
bashnano src/main/resources/application.properties
```

```properties
server.port=8000
server.servlet.context-path=/app
```

#### 4.3 `pom.xml`

```bash
bashnano pom.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>holamundo</artifactId>
    <version>1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Paso 5 — Compilar proyecto

```bash
bashmvn clean package -DskipTests
```

Esperado: `BUILD SUCCESS`.

### Paso 6 — Levantar app Spring Boot

```bash
bashjava -jar target/holamundo-1.0.jar &
```

Verificar:

```bash
bashcurl http://localhost:8000/app/
```

Respuesta esperada: `Hola Mundo desde Spring Boot!`

### Paso 7 — Configurar NGINX como reverse proxy

```bash
bashnano /etc/nginx/sites-available/default
```

Contenido:

```nginx
server {
    listen 80;

    location /app/ {
        proxy_pass http://localhost:8000/app/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Iniciar NGINX:

```bash
bashservice nginx start
```

### Paso 8 — Verificación final

```bash
bashcurl http://localhost/app/
```

Debe devolver: `Hola Mundo desde Spring Boot!`

La diferencia con el paso 6 es que ahora la petición entra por NGINX (puerto 80) y se redirige internamente al 8000 de Spring Boot.

---