#Crea tu propia nube  con VPN
##Introducción
Este proyecto tiene como objetivo configurar un servidor NAS (Network Attached Storage) utilizando Nextcloud y una VPN (Virtual Private Network) con Tailscale en una Raspberry Pi. Nextcloud te permitirá crear tu propia nube privada para almacenar y compartir archivos de manera segura, mientras que Tailscale proporcionará una conexión VPN sencilla y segura para acceder a tu NAS desde cualquier lugar.

##Requisitos
###Hardware:
* Raspberry Pi (preferiblemente modelo 4)
* Tarjeta microSD (mínimo 16GB)
* Fuente de alimentación compatible
* Conexión a internet (Ethernet o Wi-Fi)
* Almacenamiento externo (disco duro USB o SSD)
* -OPCIONAL- Monitor, mouse y teclado para la raspberry (si tienes mas conocimiento puedes configurar escritorio remoto de la raspberry).
### Software:
* Raspberry Pi OS Lite
* Nextcloud
* Tailscale
##Características
###Nextcloud:
* Almacenamiento y sincronización de archivos
* Compartición de archivos y carpetas
* Aplicaciones adicionales para mejorar la funcionalidad (calendario, contactos, etc.)
###Tailscale:
* Configuración sencilla de VPN
* Conexión segura y cifrada
* Acceso remoto a tu NAS desde cualquier dispositivo

##Pasos Generales
1. Preparación de la Raspberry Pi:
###Instalación de Raspberry Pi OS
1. 1.  Configuración inicial del sistema
descargar e intalar el software oficial de raspberry para cargar el sistema operativo en la micro sd https://www.raspberrypi.com/software/
NOTA cerrar cualquier aviso de windows durante la instalacion del software..
al finalizar conectar el teclado, monitor y mouse a la raspberry y encender.

1. 2. obtener la ip de la rapberry

metodo 1 ingresar las credenciales creadas durante la configuracion del sistema operativo e ingresar el siguiente comando -ifconfig-
metodo 2 acceder a tu router (tienes que conocer la dirección IP de tu router o de la administracion de tu red mesh)
ingresar con tus credenciales y localizar el dispositvo. tambien puedes utilizar una aplicacion en tu celular que te permita ver los dispositivos conectados a tu red.
abrir terminal cmd en windows
colocar el comando
ssh usuario@192.168.1.1

1. 3. Actualizar la raspberry
Ingresar el comando
sudo apt update && sudo apt upgrade -y
esto instalara diversos paquetes, libreria etc para tener el sistema operativo actualizado, esto puede demorar un poco.
posterior a eso tendremos que reiniciar la raspberry para que los cambios surtan efecto 
sudo reboot
esperar 1 minuto en lo que reincia antes de conectarnos por ssh nuevamente via ssh con el comando del paso 1.2

1. 4. configurar ip estatica 
Una IP estática asegura que siempre puedas acceder a tu Raspberry Pi en la misma dirección IP. Esto es especialmente importante para servicios como Nextcloud, donde necesitas una dirección fija para acceder a tus archivos y configuraciones.
es necesario tener en cuenta el rango de asignacion de IP de nuestro router ya que no podremos elegir una IP dentro de este rango ya que esta puede cambiar a consideracion del router. Dado que los routers varian en tamaño forma y marca tendras que investigar por tu cuenta ¿Cual es y como cambiarlo? para que la IP que asignes a tu Raspberry no este dentro de este rango y pueda funcionar correctamente. En mi caso configure mi router con un rango de hasta 253 ya que la ip 253 y 254 son las que utilizare para configurar la red ethernet y la red wifi de la raspberry para garantizar una conexion via wifi o cable ethernet si alguna llega a fallar.

Estos datos seran importantes mas tarde que los tengas presentes:
Direccion IP (la IP que quieres que tenga la raspberry)a fija: -En mi caso- 192.168.3.254
Estos datos si los puedes copiar y pegar:
Puerta de enlace 192.168.3.1
mascara de subred /24
Servidores DNS (servidores de google): 8.8.8.8,8.8.4.4
Tipo de conexion (segun sea tu caso): ethernet o wifi

esta configuracion la haremos con los siguientes comandos mediante el networK manager de raspbian.

* identificar el nombre de la interfaz de red ethernet 
sudo nmcli -p connection show

debes anotar el nombre en mi caso "Wired connection 1" si usaras la wifi usa el nombre preconfigured o el que te aparezca

* fijar ip
sudo nmcli con mod "Nombre_de_tu_interfaz_de_red" ipv4.addresses tu_IP/24 ipv4.method manual

en mi caso: 
sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.3.254/24 ipv4.method manual

* establecer puerta de enlace

sudo nmcli con mod "Nombre_de_tu_interfaz_de_red" ipv4.gateway IP_de_tu_router

en mi caso:
sudo nmcli con mod "Wired connection 1" ipv4.gateway 192.168.3.1

* colocar los servidores DNS
sudo nmcli con mod "Nombre_de_tu_interfaz_de_red" ipv4.dns "8.8.8.8,8.8.4.4"

en mi caso:
sudo nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8,8.8.4.4"

* reiniciar el network manager de la raspberry:

sudo nmcli con down "Nombre_de_tu_interfaz_de_red" && sudo nmcli con up "Nombre_de_tu_interfaz_de_red"

en mi caso:
sudo nmcli con down "Wired connection 1" && sudo nmcli con up "Wired connection 1"

NOTA: En este punto la conexion ssh se perdera por lo que deberas realizarla de nuevo como en los pasos iniciales, pero con la IP fija que se configuró anteriormente.

* verificar la ip fija
se realiza con el comando ifconfig y tiene que tener la que configuraste.

##2 Instalacion de paquetes:
a continuacion isntalaremos Apache poder utilizar un servidor web y que pueda funcionar Nextcloud el servicio de nube y PHP para una correcta ejecucion de la interfaz frontend con este comando, asegurate de copiarlo completo, puede demorar unos minutos.

sudo apt install apache2 libapache2-mod-php mariadb-server php-gd php-json php-mysql php-curl php-mbstring php-intl php-imagick php-xml php-zip

##3 Configuración de Nextcloud:
3. 1. configuracion de la base de datos y usuarios para nextcloud
Ingresar el siguiente comando:
sudo mysql -u root
una vez dentro de mariadb ingresar la siguiente lista de comandos:
CREATE DATABASE nextcloud;
en los siguientes comandos remplaza usuario y contraseña por los que tu quieras usar:
CREATE USER 'usuario'@'localhost' IDENTIFIED BY 'contraseña';
dar privilegios al usuario sobre la base de datos
GRANT ALL PRIVILEGES ON nombre_base_de_datos.* TO 'usuario'@'localhost';
FLUSH PRIVILEGES;
EXIT;
3. 2. Descarga de nextcloud
se descargara nextcloud en la carpeta temporal (tmp) del sitema para descomprimir los archivos necesarios y que al reiniciar la raspberry se eliminen y no usen espacio en la microSD
ir a la carpeta tmp con:
cd /tmp/
comando:
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
con el comando ls debera aparecer una carpeta llamada nextcloud la cual moveremos al directorio publico del servidor apache y que pueda ser hospedada.
sudo mv nextcloud/ /var/www/html/
entrar al directorio
cd /var/www/html/
cambiar el propietario de la carpeta nextcloud al usuario del servidor apache
sudo chown -R www-data:www-data nextcloud

4. configuracion del servidor apache 
crea el fichero de configuracion para el sitio web de nextcloud
sudo nano /etc/apache2/sites-available/nextcloud.conf
copiar las siguientes lineas de la configuracion del sitio:
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/nextcloud/

    Alias /nextcloud "/var/www/HTML/nextcloud/"

    <Directory /var/www/html/nextcloud/>
        Options +FollowSymlinks
        AllowOverride All
        
       <IfModule mod_dav.c>
	  Dav off
       </IfModule>

       SetEnv HOME /var/www/html/nextcloud
       SetEnv HTTP_HOME /var/www/HTML/nextcloud
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

para salir de nano presionar ctrl + O
ctrl+x
habilitar el sitio de nextcloud y los modulos de apache requeridos:
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite headers env dir mime setenvif ssl
reiniciar apache
sudo systemctl restart apache2

cambiar el directorio data de ubicacion publica a privada var
cd /var
crear directorio data
sudo mkdir data
ls -l data/
cambiar el propietario de Root al usuario de nextcloud
sudo chown www-data:www-data data/

## 5 ingresar a nextcloud desde el navegador en windows
ingresar la url: tu_IP/nextcloud/index.php
y crea tu  cuenta de administrador guarda bien estas credenciales (usiario y contraseña)
en el campo almacenamiento y base de datos colocar /var/data
Cuenta de la base de datos - tu usuario que creaste en el paso de crear base de datos
contraseña de la base de datos - la contraseña que ingresaste para la base de datos
nombre de la base de datos- nombre que le pusiste a la base de datos

dar click en instalar
se mostrara la siguiente pantalla al instalar.

## 6 Configuración de almacenamiento externo
una vez conectado el disco duro externo a la raspberry hay que realizar los siguientes pasos:
1. buscar el disco en el sistema mediante el siguiente comando que mostrara los discos de almacenamiento conectados:
sudo lsblk
2 formatear el disco 
sudo su
lsblk y anotar el nombre en este caso: sdb
3. desmontar el disco
sudo umount /dev/sdb1
4. revisar particiones, limpiar y crear
fdisk -l /dev/sda
fdisk /de/sda
d
w
fdisk -l /dev/sda
5. crear particion
fdisk /dev/sda
n
p
enter
enter
enter
y
w
dar formato al disco externo a ex4
mkfs.ext4 /dev/sda1
esperar puede tardar bastante
comprbar formato
blkid dev/sda1
debe mostrar ext4

creando el directorio de montaje
mkdir /mnt/disco1

montar el disco en el directorio 
lsblk
mount /dev/sda1 /mnt/disco1
configuracion del disco en nextcloud 
abrir menu de configuraciones y administracion
buscar la ultima opcion llamada sistema
habilitar extension de alamcenamineto de nextcloud 
entrar en tus a la barra de opciones derecha y aplicaciones
en la nueva barra izquierda entrar en apps activas y habilitar la aplicacion external disk support 

agregar permisos para que se puedan añadir/eliminar archivos
/mnt# chown -R www-data:www-data /mnt/disco1
verificar en nextcloud si se pueden crear carpetas o subir archivos

guardando configuraciones del montaje disco 
hay que configurar la raspberry para que guarde el montado del disco externo ya que si en este momento se reinicia o se apaga el discos se desmonta y no podremos acceder a la informacion.
hay que configurar un fichero del sistema para que el montaje de discos se haga automatico al encender la raspberry
sudo nano /etc/fstab
/dev/sda1       /mnt/disco1     ext4    defaults        0       0
ctrl + o
enter
ctrl+ x
reiniciar la raspberry, esperar dos minutos y recargar nextcloud en el navegador


Configuración de Tailscale:
Instalación y configuración de Tailscale
Conexión de dispositivos a la VPN
Pruebas y Verificación:
Verificación del acceso a Nextcloud
Pruebas de conexión VPN con Tailscale
Conclusión
Al finalizar este proyecto, tendrás un NAS funcional con Nextcloud y una VPN segura con Tailscale, todo corriendo en una Raspberry Pi. Esto te permitirá tener control total sobre tus datos y acceder a ellos de manera segura desde cualquier lugar.