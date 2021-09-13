---
layout: post
title:  "Publicando nuestro sitio Web"
date:   2021-09-08 21:30:59 -0700
categories: jekyll ruby ubuntu docker
tags:
- Creador de contenido
- docker
- Ubuntu
---

En esta entrada vamos a aprender un poco de Docker, el cual usaremos para levantar un sitio web usando nginx y letsencrypt para tener un servidor web y un certificado ssl.
Para este tutorial no es necesario tener grandes conocimientos de Docker ni de nginx, pero si quieres hacer configuraciones un poco más complejas tendrás que investigar por tu cuenta o puedes dejar en la sección de los comentarios.

# Prerrequisitos 
* Un sitio web (en este tutorial usaremos un sitio creado con Jekyll)
* Una IP publica 
* un nombre de dominio
* Un servidor con Ubuntu 
* Docker
* Docker compose

# Construyendo nuestro sitio con Jekyll
En el post anterior aprendimos a crear sitios con jekyll pero solo sabemos cómo probarlo en nuestro entorno local. Para hacer un buid de nuestro sitio es algo muy fácil que no da tanto contenido para hacer una entrada. 
Nos dirigimos a la carpeta donde tenemos el proyecto y ejecutamos
```
bundle exec Jekyll build
```
Con este comando vamos a actualizar el build de nuestro sitio y se convertirá en código htm, el cual ya podremos montar en un servidor web, el tenido lo encontraremos en `_site` toda esa carpeta la mandaremos a nuestro servidor a mí me gusta usar scp para mandar información a los servidores
```console
scp ./_site MIUSUARIO@IP_DE_MI_SERVIDOR:/RUTA
```
En otra entrada aprenderemos a hacer integraciones con github accions para automatizar este proceso.

# Publicando nuestro sitio en un servidor
Para este punto ya debemos tener nuestro servidor con una IP publica, nuestro dominio apuntando a la IP de nuestro servidor y con Docker instalado.
Hacemos una carpeta en nuestro servidor yo la llamare webserver, movemos la carpta _site y la renombramos a www.

 ```console
mkdir webserver
mv _site webserver/www
```
Ahora tenemos una estructura de carpetas con solo nuestro sitio en la carpeta www

```
▾ webserver/
└─▾ www
  └── index.html
  └── … todas las demás carpetas de nuestro sitio 
```

Vamos a crear nuestro archivo `docker-compose.yml` en el cual tendremos la configuración de nuestro sitio.
```console
vim docker-compose.yml
```
escribimos esta configuración en el archivo 


```yaml
www:
  image: nginx
  restart: always
  volumes:
    - ./www:/usr/share/nginx/html:ro
  expose:
    - "80"
  environment:
    - VIRTUAL_HOST=dotlog.xyz,www.dotlog.xyz
    - LETSENCRYPT_HOST=dotlog.xyz,www.dotlog.xyz
    - LETSENCRYPT_EMAIL=luise.gstlm@gmail.com
```
Levantamos la imagen de docker 

```console
sudo docker-compose up 
```

## Usar un proxy

Entramos con un navegador a la url de nuestro dominio o la ip pública del servidor y ya veremos el sitio.

Con esa simple configuración ya podemos acceder a nuestro sitio usando nuestro dominio, pero nosotros queremos que a nuestro sitio se entre por ssl y salga el candadito verde en nuestra pagina.

Para conseguir usar el ssl vamos a necesitar un nginx-proxy, lo tenemos que agregar a nuestro archivo `docker-compose.yml` 
```yaml
nginx-proxy:
  image: jwilder/nginx-proxy
  restart: always
  ports:
    - "80:80"
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock
    - ./logs:/var/log/nginx

www:
  image: nginx
  restart: always
  volumes:
    - ./www:/usr/share/nginx/html:ro
  expose:
    - "80"
  environment:
    - VIRTUAL_HOST= tusitio.com,www.tusitio.com

 ```
Agregamos un servicio llamado `nginx-proxy`  y usa la imagen `jwilder/nginx-proxy` exponiendo el puerto 80 de la maquina de docker a nuestro servidor, y la etiqueta `volumes` hacemos proxy del servicio `www`, y el otro volumen que agregamos es para pasar los logs del servicio `nginx-proxy` a nuestra carpeta. También editamos el servicio `www` quitando la `etiqueta ports` y solo dejando la etiqueta `expose` con el puerto 80




con esto también podemos levantar el docker con `sudo docker-compose up` pero aún no tenemos habilitado el certificado ssl.

## habilitando SSL
Para habilitar el ssl usaremos otra imagen, la de letsencrypt, vamos a editar de nuevo el archivo `docker-compose.yml` ya para terminar la configuracion 

```yaml
nginx-proxy:
  image: jwilder/nginx-proxy
  restart: always
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock
    - ./logs:/var/log/nginx
    - certs:/etc/nginx/certs:ro
    - confd:/etc/nginx/conf.d
    - vhostd:/etc/nginx/vhost.d
    - html:/usr/share/nginx/html
  labels:
    - com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy

letsencrypt:
  image: jrcs/letsencrypt-nginx-proxy-companion:stable
  restart: always
  volumes:
    - certs:/etc/nginx/certs:rw
    - confd:/etc/nginx/conf.d
    - vhostd:/etc/nginx/vhost.d
    - html:/usr/share/nginx/html
    - /var/run/docker.sock:/var/run/docker.sock:ro

www:
  image: nginx
  restart: always
  volumes:
    - ./www:/usr/share/nginx/html:ro
  expose:
    - "80"
  environment:
    - VIRTUAL_HOST= tusitio.com,www.tusitio.com
    - LETSENCRYPT_HOST= tusitio.com,www.tusitio.com
    - LETSENCRYPT_EMAIL=tu_correo
```

Con esta configuración tu la puedes replicar para tu sitio y va a funcionar solo cambia en las etiquetas environments por tus datos y listo, pero vamos a ver que hicimos en esta ultima parte

1.	Agregamos el servicio letsencrypt que usa la imagen jrcs/letsencrypt-nginx-proxy-companion:stable.
2.	Agregamos a la etiqueta ports del servicio nginx-proxy el puerto 443 y agregamos volumenes para el certificado, los virtualhost y el html de la pagina.
3.	Al servicio www le agregamos dos environment nuevos para la configuración de letsencrypt.



Corremos nuestro servicio con -d para que corra en segundo plano
```console
sudo docker-compose up -d
```
Conseguimos una estructura de carpetas 

```tree
▾ webserver/
├─▾ www
│   ├── index.html
│   └── … todas las demás carpetas de nuestro sitio 
├─▾ logs
│   ├── access.log
│   └── error.log
└── docker-compose.yml

```
# Conclusión 
Con docker es muy simple levantar un sitio web de una forma muy fácil y rápida, esto nos va a servir en caso de que tengas que publicar una página de documentación o un blog.

# Conclusión 
Con docker es muy simple levantar un sitio web de una forma muy fácil y rápida, esto nos va a servir en caso de que tengas que publicar una página de documentación o un blog.

# Referencias 

* [https://hub.docker.com/\_/nginx](https://hub.docker.com\_/nginx)
* [https://hub.docker.com/r/jrcs/letsencrypt-nginx-proxy-companion/](https://hub.docker.com/r/jrcs/letsencrypt-nginx-proxy-companion/)
* [https://hub.docker.com/r/jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy)

