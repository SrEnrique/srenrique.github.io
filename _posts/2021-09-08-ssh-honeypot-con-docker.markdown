---
layout: post
title:  "SSH Honeypot con docker"
date:   2021-09-08 21:30:59 -0700
categories: ssh hoynepot docker ubuntu
---
Según Wikipedia un honeypot o señuelo, es una herramienta de la seguridad informática dispuesto en una red o sistema informático para ser el objetivo de un posible ataque informático, y así poder detectarlo y obtener información de este y del atacante.

La característica principal de este tipo de programas es que están diseñados no solo para protegerse de un posible ataque, sino para servir de señuelo invisible al atacante, con objeto de detectar el ataque antes de que afecte a otros sistemas críticos. El honeypot, sin embargo, puede estar diseñado con múltiples objetivos, desde simplemente alertar de la existencia del ataque u obtener información sin interferir en el mismo, hasta tratar de ralentizar el ataque y proteger así el resto del sistema.

De esta forma se tienen honeypots de baja interacción, usados fundamentalmente como medida de seguridad, y honeypots de alta interacción, capaces de reunir mucha más información y con fines como la investigación.
Vamos a levantar un honeypot de un servicio ssh el cual estará capturando y registrado los usuarios y contraseñas que se estén usando para intentar ingresar a nuestro servidor, por ejemplo, en un ataque de fuerza bruta.

# Cambiar puerto por defecto de ssh


Editar archivo de configuración de ssh, para cambiar el puerto por defecto y dejar disponible el puerto 22 para el honeypot 

```bash
sudo vim /etc/ssh/sshd_config
```
Agregar la siguiente linea

```bash
Port 222
```
Reiniciar ssh

```bash
sudo service ssh restart
```



# Clonar el repositorio


clonar

```bash
git clone https://github.com/SrEnrique/SrEnrique-docker-ssh-honeypot.git
```

Ejecutar Docker 

```bash
cd docker-ssh-honypot
sudo docker-compose up -d
```

Leer los logs de Docker 

```bash
sudo docker logs -f $(sudo docker ps -f name=ssh_honeypot_webapp_1 -q)
```


## Referencias 

[hub.docker.com wildwildangel/ssh-honeypotd](https://hub.docker.com/r/wildwildangel/ssh-honeypotd)