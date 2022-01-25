## Hardening Apache

### Instalación Apache

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

### Configuraciones globales

Dentro del directorio /etc/apache2 encontraremos varios directorios y ficheros que nos permiten configurar tanto de forma global como específica nuestro servidor:

**apache2.conf:** Configuración principal des de donde se cargarán todos los ficheros necesarios cuando se inicie el servidor web.

**ports.conf:** Determina los puertos que escucha para las conexiones entrantes.

**mods-available, conf-available, sites-available:** Configuración específica para gestionar módulos, configuración global y hosts virtuales de forma separada.

**mods-enabled, conf-enabled, sites-enabled:** Enlaces simbólicos desde *-available (mods, conf y sites).

**envvars:** Contiene algunas variables de entorno usadas en apache2.conf.

#### Usuarios y grupos

Tenemos que configurar un usuario y grupo no provilegiado para el servidor web editando el fichero /etc/apache2/apache2.conf y buscand la parte de "User" y "Group".
```
- User ${APACHE_RUN_USER}
- Group ${APACHE_RUN_GROUP}
```
Los valores de estas variables de entorno se establecen en /etc/apache2/envvars
```
- export APACHE_RUN_USER=<usuario>
- export APACHE_RUN_GROUP=<grupo>
```
Por ejemplo www-data. Lo normal es que ya tengamos configurados estos valores con el Usuario y Grupo www-data, pero debemos verificarlo siempre.

De esta manera al arrancar el servidor web lo hara con el usuario que le hayamos indicado, ya que si arrancamos el servidor con usuario privilegiado como root, corremos el riesgo de, ante un posible ataque exitoso, otorgar permisos de administrador al atacante.

Con un *ps auwwfx | grep apache* podemos ver el usuario que esta ejecutando el servicio Apache.

#### Ocultación de versiones ¿Cuales son las fases de un ataque?

+ Reconocimiento: el atacante busca información sobre la empresa. Generalmente, empieza a investigar los datos de la organización publica en abierto para tratar de averiguar qué tecnologías utiliza e interactuar con el correo electrónico y las redes sociales.
+ Preparación: el ciberdelincuente prepara su ataque hacia un objetivo específico.
+ Distribución: en esta fase se produce la transmisión del ciberataque. De nuevo, la formación de los trabajadores y el conocimiento de estas técnicas es la mejor forma de prevenirlo.
+ Explotación: el atacante compromete el equipoinfectado y su red. Esto lo suele conseguir explotando una vulnerabilidad para la que exista un parche de seguridad pero no esté operativo. Es importante contar con soluciones de seguridad y un antivirus actualizado para evitar que tu empresa quede expuesta.
+ Instalación: el atacante instala el malware o roba las credenciales de la víctima, entre otras posibilidades. Implementar una correcta formación sobre el tema es la mejor formade evitarlo, así como las medidas técnicas que van desde la seguridad de la información, hasta la privacidad de los datos, pasando por los controles de acceso.
+ Comando y control: durante esta, el atacante ya cuenta con el control del sistema. Puede llevar a cabop acciones maliciosas como robar información confidencial, extraer datos de acceso y capturas de pantalla, instalar programas y conocer aún más a la víctima y su red.
+ Acciones sobre los objetivos: esta es la fase final, en la que el atacante consigue los datos que necesita y expande el ciberataque, iniciando a la vez el ciclo otra vez y repitiendo etapas.

En definitiva, la mejor forma de romper esta cadena es la concienciación en la empresa, la formación del personal y contar con las soluciones de seguridad adecuadas. De esta manera se eliminará la vulnerabilidad y será mucho más segura.

#### Exposición mínima de módulos

En Apache, los módulos pueden estar compilados estáticamente o cargqados de forma dinámica.

Un módulo es una extensión que añade funcionalidad a través de la configuración en Apache, concretamente añadiendo directivas propias del módulo.

En instalaciones por defecto es habitual encontrar módulos activos que no se usan o no nos insteresan.

Siempre es recomendable desactivar cualquier módulo que no esté en uso. Reducirá la carga y la superfície de exposición de un ataque.

Para activar o desactivar módulos usaremos en línia de comandos las utilidades a2dismod y a2enmod añadiendo como argumento el nombre del módulo.

Para consultar la lista de módulos, ya sean estáticos o dinámicos, disponemos en línia de comandos de apache2ctl con el argumento -M mayúscula. Seguido de recargar la configuración con "sudo systemctl reload apache2" para que los cambios aplicatos tengan efecto.

### Creación de virtualhost

Cogiendo como ejemplo el /etc/apache2/sites-enabled/000-default.conf
```
<VirtualHost *:80>
    ServerName localhost
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    <Directory /privado>
        Options +FollowSymlinks
        AllowOverride None
    </Directory>
    LogLevel info
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### Configuración múltiple y contexto: directiva options

Nuestro directorio raíz es /var/www/html

La directova "Directory" se ha definido para el recurso "privado" que se encuentra en el directorio raíz.

Si navegamos a través de http://localhost/privado/ accederemos a modo índice web a todos los ficheros.

¿Como podemos evitar esto?
    + Con la directiva "Options -Indexes"
    + Deshabilitando el módulo autoindex

```
<VirtualHost *:80>
    ServerName localhost
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    <Directory /var/www/html/privado>
        Options -indexes +FollowSymlinks
        AllowOverride None
    </Directory>
    LogLevel info
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Nuestro directorio raíz es /var/www/html/

"Directory" es la directiva que hemos definido para el recurso "privado" que se encuentra en el directorio raíz.

Ahora si navegamos a través de http://localhost/privado/ nos mostrará un mensaje HTTP 403 Forbidden.

Los ficheros .htaccess que residen en drectorios proporcionan el mismo resultado

#### Restringiendo acceso al contenido: directiva Auth y Require. Aplica la configuración para autenticar el acceso mediante digest a uno de los directorios de tu virtualHost

+ Creamos un usuario y contraseña
    sodo htpasswd -c passwords albert
+ Añadimos directivas Auth*
```
<Directory /var/www/html/privado>
    Options +FollowSymLinks
    AllowOverride None

    AuthType Basic
    AuthName "Acceso Restringido"
    AuthBasicProvider file
    AuthUserFile "/etc/apache2/passwords"
</Directory>
```
+ Reiniciamos el servidor web+
    systemctl restart apache2

#### Ficheros .htaccess ¿Para qué sirven?

El archivo .htaccess es un archivo oculto que se utiliza para configurar funciones adicionales para sitios web alojados en el servidor web Apache.

Estos permiten personalizar la configuración de directivas y parámetros que se definen en el fichero principal de configuración de Apache. Deben colocarse dentro de un directorio donde se pretende tenga efecto. Estos ficheros están protegidos desde la directiva del fichero principal, dando un 403 Forbidden en caso de acceder directamente a ellos.

Su flexibilidad de configuración les proporciona alta probabilidad de usos incorrectos.

### ¿Cómo podemos evitar el hotlinking? 

El hotlinking es una práctica empleada por propietarios de una página web para usar el contenido de otra página web, concretamente imágenes, vídeos o documentos alojados en la web de origen, sin pedir permiso, sin pagar licencias y empleando para ello el mínimo esfuerzo posible.

Afortunadamente, existen diferentes maneras para prevenir el hotlinking de los contenidos de nuestra web.

+ Con un CDN que incluya Protección de Hotlinking
+ Permitiendo la Protección Hotlinking en Apache
+ Habilitar Protección de Hotlinking en NGINX
+ Plugins de WordPress que nos pueden ayudar
+ Deshabilitando Clic Derecho en WordPress
+ Renombrando archivos
+ En la configuración de nuestro cPanel

#### Configuración HTTPS mediante Let's Encrypt o OpenSSL. Crea los certificados para que tu virtualHost sea seguro, y obligatoriamente los accesos sean por HTTPS

### Módulo "mod_security" ¿Qué es mod_security?

Es un módulo de seguridad de Apache, actúa como firewall de aplicaciones web (WAF) y su trabajo es filtrar y bloquear las solicitudes HTTP sospechosas, pudiendo bloquear ataques de fuerza bruta, vulnerabilidades de cross scripting (XSS), ataques por inyección SQL (SQLi), etc.

mod_security está activo en todos nuestros servidores Linux por defecto. Aunque no es posible deshabilitarlo completamente por razones de seguridad, este módulo permite añadir excepciones mediante el fichero .htaccess en el caso de que trate de un falso positivo.

#### Realiza un ataque DoS mediante Metasploit (Slowloris) y comprueba que efectivamente el servidor está inaccesible

#### Clona e instala las reglas recomendadas OWASP. Habilita mod_security

### Reglas para detectar SQLInjection

### Realiza de nuevo el ataque DoS y comprueba que el servidor está accesible