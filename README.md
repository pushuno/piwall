# Piwall
---
Armado de un Wall de Video con Raspberry Pi

El objetivo de este documento es compartir mi experiencia armando un wall de video con Piwall y Raspberry's con el fin de ahorrar tiempo a quienes quieran probarlo, hablaré sobre los inconvenientes que tuve durante su armado y sus soluciones, las ventajas y limitaciones.
Nada de lo descripto es verdad absoluta, solo cuenta mi experiencia hasta el momento con el sistema.

TODO EL SOFTWARE UTILIZADO ES LIBRE y GRATUITO

Empezamos?

### Entorno
---
Armar un Wall de Video siempre me llamó la atención como a varios supongo, cada día que pasa las pantallas son mas grandes y con bordes mas finos lo que hace mas interesante el asunto. 
Se necesitará una Raspberry Pi por cada pantalla que querramos agregar al Wall, en este caso yo utilicé dos pantallas FullHD de 32" Samsung.

### Ventajas
---
- Menos cables
En el pasado la única forma de hacer un wall de video era con costosos splitters y muchos cables. 
Los unicos cables que necesitaremos son por cada raspberry 

### Limitaciones
---
- La conexión mediante wifi por más que utilicemos redes AC no es viable.
Realicé varias pruebas y utilizando UDP como vamos a usar se pierden paquetes y se ven franjas verdes
- Pwomxplayer no es muy claro con los mensajes de error
ante un error crea un log que no da mucha información que ayude a solucionar el problema
- Para crear un video con Premiere Pro a la mejor resolución y calidad posible y que funcione y fluido hay que tener una configuración específica de codecs.
- No es posible utilizar pantallas en ángulos no rectos, con esto me refiero a que se pueden colocar las pantallas giradas pero siempre verticales u horizontales. No encontré configuración posible para colocarlas a 45º como era mi idea en un principio, para esto la única alternativa es utilizar un software pago (No son baratos y son por suscripción por lo menos para lo que yo pretendía hacer). Si te interesa esto podés buscar por acá www.info-beamer.com
- PwOMXPlayer no soporta el comando --loop como si soporta OMXPlayer. 


### Requisitos
---
- Una Raspberry Pi por cada pantalla que querramos agregar al wall.
En mi caso utilicé una Raspberry Pi 3 Model B y una Raspberry Pi 4 Model B de 2GB de Ram, tengo entendido que se puede también con un modelo de Pi Zero pero no lo probé.
Puede utilizarse una pc con Linux o MacOs además para procesar el video o puede ser controlado por una Raspberry. Dependiendo de el tipo de video que se quiera mostrar.
A más resolución yo utilizaría un equipo mas dedicado para la transmisión de video.

- Pantallas.
Se puede usar cualquier pantalla que tenga HDMI o las pantallas chiquitas que venden para Raspberry que van en el DSI Display Port.

- Memoria
Memorias de al menos 16GB Clase 10 Micro SD para las Raspis


### Configurando los Clientes
---
- Descargar Raspbian
Lo primero es descargar la imágen de Raspbian que es una versión de Debian adaptada para Raspberry.
Se puede descargar la versión con interfaz gráfica o sin interfaz gráfica, yo utilicé con interfaz ya que la idea mia es que cuando no esté reproduciendo video utilizarlas como un cuadro donde el fondo de pantalla muestre una imagen partida. Para el Wall puede utilizarse cualquiera de las dos versiones, eso si debe ser la última disponible. A la fecha la última es "Raspbian Buster" y se puede descargar de acá https://www.raspberrypi.org/downloads/raspbian/

- Cargar la imágen en las Raspis
Para cargar la imágen utilicé en MacOs Balena Etcher, en windows se puede usar cualquier programa que permita grabar una imágen en una memoria se puede descargar de acá https://www.balena.io/etcher/

- Habilitar SSH, VNC
Preferencias > Configuración de Raspberry Pi > Interfaces

- Instalar TFTP en servidor
Si el servidor es una Raspberry usaremos el protocolo FTP para copiar videos en el.
Mediante Interfaz: Preferencias > Add / Remove Software > tftp > Apply
Por línea de comando: sudo apt-get install tftp

- Fijar IP
O la fijamos por MAC desde el router o la fijamos desde la configuración de DHCP de las Raspberrys.
Ya que no estarán conectadas permanentemente a internet lo configuraremos desde las Raspberrys.
Para asignar las IPs de los equipos usaremos la ip 100 para el servidor y las proximas para los clientes.

Ej. 192.168.0.100 (Servidor) y 192.168.0.101 (Cliente 1) 192.168.0.102 (Cliente 2) etc.

Para Fijar la Ip por línea de comandos vamos a
sudo nano /etc/dhcpcd.txt

buscamos donde dice "Example static IP configuracion"
descomentamos las lineas borrando el # y ponemos la ip que queremos para el equipo
Luego Reiniciamos
Verificamos que la ip se haya aplicado bien por linea de comandos
ifconfig

- Agregar memoria a GPU
Preferencias > Configuración de Raspberry Pi > Rendimiento
y le ponemos 128 MB

- Deshabilitar ahorro de energía
Editar
/etc/xdg/lxsession/LXDE-pi/autostart

agregar
@xset s noblank
@xset s off
@xset -dpms

- Actualizar
sudo apt-get update

- Instalar pwlibs y pwomxplayer
pwomxplayer es una adaptación de OMXPlayer preparada para facilitar la configuración del wall

wget http://dl.piwall.co.uk/pwlibs1_1.1_armhf.deb
sudo dpkg -i pwlibs1_1.1_armhf.deb

wget http://dl.piwall.co.uk/pwomxplayer_20130815_armhf.deb
sudo dpkg -i pwomxplayer_20130815_armhf.deb

- Cambiar nombre de el equipo
Lo cambiaremos haciendo referencia a la ip. Ej. 192.168.0.101 => pi101
Para ello vamos a editar
sudo nano /etc/hostname
y cambiamos el nombre y 
sudo nano /etc/hosts
y cambiamos donde dice raspberry a el nombre del equipo nuevo

- Creamos un archivo en /home/pi

sudo nano /home/pi/.piwall

con el siguiente contenido

[pushuno_wall]
width=1560 #suma del total de anchos
height=720 #suma del total de altos
x=0
y=0

[pi200] #nombre de el equipo
wall=pushuno_wall #nombre del wall a fraccionar
width=1280 #ancho de pantalla en mm (solo lo visible, sin marco)
height=720 #alto de pantalla en mm (solo lo visible, sin marco)
x=0 #posicion de donde empieza el recorte en x
y=0 #posicion de donde empieza el recorte en y

[pi201]
wall=pushuno_wall
width=1280
height=720
x=1281 #sumar los mm de bordes de pantalla para ver donde empiezan todos
y=0

[pushuno_wall]
pi1 = pi100
pi2 = pi101

Para los valores de X e Y hay que tener en cuenta si el wall es horizontal o vertical. (Es la distancia que queda entre que termina la imágen de una pantalla e inicia la de la próxima)

- Creamos el archivo .pitile

sudo nano /home/pi/.pitile

con el siguiente contenido
[tile]
id = pi100 #nombre del equipo

- probar reproducción de video local
copiamos un video por ftp o con un pendrive a la raspberry y ejecutamos

pwomxplayer video.avi

si el video se reproduce bien podemos salir con (Ctl+C)
Si tira error probar ejecutando lo siguiente 

Sudo apt-get upgrade
Sudo rpi-update

### Configuración del Servidor
---
si el servidor también es cliente ejecutar lo anterior también.

- Instalamos ffmpeg o avconv

sudo apt-get install ffmpeg

- Instalamos PwOMXPlayer

wget http://dl.piwall.co.uk/pwomxplayer_20130815_armhf.deb
sudo dpkg -i pwomxplayer_20130815_armhf.deb

- Habilitar SSH, VNC
Preferencias > Configuración de Raspberry Pi > Interfaces

- Instalar TFTP en servidor
Si el servidor es una Raspberry usaremos el protocolo FTP para copiar videos en el.
Mediante Interfaz: Preferencias > Add / Remove Software > tftp > Apply
Por línea de comando: sudo apt-get install tftp

- Fijar IP
O la fijamos por MAC desde el router o la fijamos desde la configuración de DHCP de las Raspberrys.
Ya que no estarán conectadas permanentemente a internet lo configuraremos desde las Raspberrys.
Para asignar las IPs de los equipos usaremos la ip 100 para el servidor y las proximas para los clientes.

Ej. 192.168.0.100 (Servidor) y 192.168.0.101 (Cliente 1) 192.168.0.102 (Cliente 2) etc.

Para Fijar la Ip por línea de comandos vamos a
sudo nano /etc/dhcpcd.txt

buscamos donde dice "Example static IP configuracion"
descomentamos las lineas borrando el # y ponemos la ip que queremos para el equipo
Luego Reiniciamos
Verificamos que la ip se haya aplicado bien por linea de comandos
ifconfig

- Copiamos en /home/pi el archuvo .piwall que generamos para los clientes que debe ser igual para todos

### Primeras Pruebas
---

- En el/los clientes ejecutamos

pwomxplayer -A udp://239.0.1.23:1234?buffer_size=1200000B

-A dice que tome de .piwall la configuración que corresponde según el .pitile que tiene la maquina
también podemos ekecutar 

pwomxplayer -R pi100 udp://239.0.1.23:1234?buffer_size=1200000B

donde -R pi100 fuerza a que tome la configuración de .piwall correspondiente al equipo pi100, o podria ejecutar

pwomxplayer -R pi udp://239.0.1.23:1234?buffer_size=1200000B

para el perfil creado en .piwall donde reproduce lo mismo en todas las pantallas.

- En el servidor copiar algún video mp4 o mpeg (luego de probarlo con PwOMXPlayer u OMXPlayer)
Todos los clientes deben estar a la espera de recibir video antes de ejecutar lo siguiente.
Ejecutar

ffmpeg -i 1080.mpg -an -f h264 udp://239.0.1.23:1234

-i establece el input (nombre del archivo a abrir)
-f establece el output y formato h264 (para que sea liviano) y salimos por broadcast udp (Esta dirección y puerto es siempre igual)
-an establece no enviar audio, si lo sacamos envia audio a todos los clientes siempre que el codec de audio del video sea reconocido por el reproductor.

### Control de todos los clientes a la vez
---
- Con SSH habilitado e instalando pssh

Sudo apt-get install pssh 

- crear una clave RSA en el servidor

ssh-keygen -t rsa -C "your@emailaddress.com"

- crear un archivo de hosts con los datos de los clientes 

sudo nano /home/pi/wall_hosts

y el contenido

pi@192.168.0.100
pi@192.168.0.101

- copiar el archivo de .ssh/id_rsa.pub a los clientes como .ssh/authorized_keys

- Controlar los clientes desde el servidor 

pssh -h wall_hosts -t 0 pwomxplayer -A udp://239.0.1.23:1234?buffer_size=1200000B

-h wall_hosts le dice cual es el archivo que contiene la lista de clientes
-t 0 deshabilita el tiempo maximo de ejecución ya que vamos a reproducir videos
luego de eso pasamos el comando que queremos ejecutar en todos los clientes. También podría 

pssh -h wall_hosts -t 0 sudo shutdown now

para apagar todos los clientes


### Sobre mi
---
Mi nombre el Lucas Febbroni, Soy Ingeniero en Informatica recibido de la Universidad de Palermo.
Apasionado por la tecnología, la integración de proyectos, Iot y toda clase de desarrollos.
Soy desarrollador en el Congreso de la Nación Argentina y desarrollo aplicaciones de todo tipo por mi cuenta.
info@pushuno.com
www.pushuno.com




