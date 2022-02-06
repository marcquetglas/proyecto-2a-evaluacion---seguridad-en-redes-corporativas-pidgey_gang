## Incorporación servicio SSH

SH es un protocolo que garantiza que tanto el cliente como el servidor remoto intercambien informaciones de manera segura y dinámica. El proceso es capaz de encriptar los archivos enviados al directorio del servidor, garantizando que las alteraciones y el envío de datos sean realizados de la mejor forma.

### Instalación del servidor SSH
```
apt install openssh-server
```

### Configuración hardenind SSH

#### Configuración de sincronización de timepo
Para la sincronización de tiempo utilizaremos systemd-timesyncd:
```
sudo apt-get install systemd-timesyncd
```
+ Implementa solo la parte del cliente del protocolo SNTP.
+ Viene precompilado en la versión de Ubuntu.
+ Se ejecuta con mínimos privilegios.
+ Se ejecuta solo cuansdo hay conectividad.

Para habilitarlo se debe ejecutar el siguiente comando:
```
*Activamos el servicio:*
systemctl enable systemd-timesyncd.service

*Editamos el fichero de configuración:*
nano /etc/systemd/timesyncd.conf

*Y añadimos las siguientes línias:*
NTP=0.ubuntu.pool.ntp.org 1.ubuntu.pool.ntp.org 02.ubuntu.pool.ntp.org
FallbackNTP=ntp.ubuntu.com 3.ubuntu.pool.ntp.org
RootDistanceMaxSec=1

*Reiniciamos el servicio para que la modificación tenga efecto:*
sysremctl start systemd-timesyncd.service
```

#### Servicios a desinstalar
+ ***X Windows System:*** sistema gráfico de ventanas típico de cualquier sistema de escritorio.
+ ***Avahi:*** sistema de descubrimiento de nuevos servicios en la red.

Ambos servicion no vienen instalados por defecto, por lo que si no los hemos instalado previamente no netesitamos realizar ninguna acción adicional. En casop de haberlos instalados, para desinstalarlos debemos ejecutar:
```
apt purge xserver-xorg
systemctl --now disable avahi-daemon
```

+ Todo servicio no relacionado con el objetivo
Para ver los servicios instalados debemos ejecutar:
```
service --status-all
```
Y aparecerá una lista con los servicios instalados y cuales se están ejecutando. Deberemos revisar los servicios existentes y deshabilitarlos o desinstalarlos directamente.

#### Configuración del sistema de ficheros
La configuración del sistema de ficheros se puede realizar al instalar el sistema o una vez instalado modificando las particiones existentes.

Se recomienda realizarlo durante la instalación del mismo para evitar tener que mover datos críticos y sectores especiales del disco, y evitar la posibilidad de corromper el sistema de ficheros.

Se recomienda tener particiones separadas para los siguientes puntos de montaje:
+ ***/tmp***: directorio accesiblew y modificable por todos los usuarios del sistema y es utilizado para almacenamiento temporal. El tamaño remoendado es de 1 GB.
+ ***/var***: directorio usado por los demonios y otros servicios del sistema para almacenar temporalmente datos dinámicos, algunos de los directorios creados pueden tener permisos de acceso y modificación para todos los usuarios por lo que también es recomendable poder tener una configuración propia independiente del resto. El tamño recomendado es de 10 GB.
+ ***/var/tmp***: directorio accesible en modo lectura, escritura y ejecución por todos los usuarios, y por lo tanto, los motivos de tener una partición independiente son los mismos. El tamaño recomendable es de 1 GB.
+ ***/home***: directorio que además de evitar el consumo de recuyrsos por parte de los usuarios locales, se debe restringir que acciones pueden realizar los usuarios en su directorio particular. El tamaño recomendado es de  10 GB.
+ ***SWAP***: partición adicional de intercambio de ficheros. El tamaño recomenrdado es de 2 GB.

##### Configuración de fstab
En el fichero /etc/fstab se encuentra la configuración inicial de carga de todas las particiones al inicio del sistema con las opciones definidas. Desde el punto de vista de seguridad, es necesario establecer una serie de opciones en las particionespara evitar el uso  de estas por parte de usuarios malintencionados.

En las particiones utilizadas para almacenar ficheros de forma temporal es necesario configurarlas coin las opciones nodev, nosuid y noexec, de forma que no se puedan desmontar sistemas de ficheros en estas particiones, ni se puedan ejecutar ficheros con el bit SUID activado y que no se permita ejecutar binarios directamente. Estas particiones son /tmp y /var/tmp.

Además, la partición /home se debe configurar con el modificador nodev y se debe configurar el dispositivo /dev/shm como nodev, nosuid y noexec.

/dev/shm es yb espacio de memoria reservado para el intercambio de datos entre aplicaciones por lo que debe tener las mismas propiedades que cualquier sistema de ficheros temporal.

#### Configuración de red
Vamos a partir de la premisa de que nuestro servidor no va a tener funciones de router y va a estar en una red IPv4.

Las configuraciones se van a centrar sobre todo en no permitir que el servidor actúe como enrutador de tráfico y en verificar el origen y destino de la comunicación, así como en registrar las acciones que puedan ser sospechosas de ser algún tipo de ataque.
29:40

#### Configuración del firewall


#### Actualizaciones de software





### Escaneo de vulnerabilidades mediante Nessus


### Explotar vulnerabilidades


### Asegurar el servidor SSH


### Segundo escaneo de vulnerabilidades y comprobación de seguridad

