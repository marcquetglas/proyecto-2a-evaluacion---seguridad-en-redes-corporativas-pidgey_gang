# Hardening Apache

## Instalación Apache

En la terminal de nuestra máquina con Ubuntu, ejecutamos los siguientes comandos:
```
apt-get update
apt-get install apache2
```
Podemos comprobar que tenemos el servicio Apache funcionando con:
```
service apache2 status
```
E incluso podemos introducir la dirección IP de nuestra máquina en el buscador, de esta manera veremos la página de Apache por defecto (index.html).

Lo primero que deberemos hacer es acceder a la ruta /var/www/html y eliminaremos el fichero index.html, ya que esta página incluye rutas y configuraciones que exponen nuestra configuración tanto de sistema operativo como de servidor web
```
cd /var/www/html
rm index.html
```

## Configuraciones globales

Dentro del directorio /etc/apache2 encontraremos varios directorios y ficheros que nos permiten configurar tanto de forma global como específica nuestro servidor:

apache2.conf: Configuración principal des de donde se cargarán todos los ficheros necesarios cuando se inicie el servidor web.

**ports.conf:** Determina los puertos que escucha para las conexiones entrantes.

mods-available, conf-available, sites-available: Configuración específica para gestionar módulos, configuración global y hosts virtuales de forma separada.

mods-enabled, conf-enabled, sites-enabled: Enlaces simbólicos desde *-available (mods, conf y sites).

envvars: Contiene algunas variables de entorno usadas en apache2.conf.

### Usuarios y grupos

