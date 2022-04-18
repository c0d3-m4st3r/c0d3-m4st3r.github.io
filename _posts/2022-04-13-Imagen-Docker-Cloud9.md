---
title: Crear contenedor para implementar el IDE de Amazon, Cloud9, en nustro propio servidor
published: true
---
Amazon nos provee de un IDE en la nube muy completo llamado Cloud9. Con este IDE en la nube podremos trabajar codo a codo con nuestros compañeros de manera remota debido a que cada cambio que haga nuestro compañero lo veremos al instante lo que hace que sea un IDE pensado para el teletrabajo. Cloud9 es capaz de compilar y ejecutar nuestros proyectos en la nube y es compatible con numerosos lenguajes, entre los que se encuentran C, C ++, PHP, Ruby, Perl, Python, JavaScript con Node. js y Go.

Para poder acceder a este entorno de desarrollo, debemos tener una cuenta creada en Amazon Web Services. Una vez tenemos la cuenta creada, iremos al enlace de cloud9 <https://console.aws.amazon.com/cloud9/>.

Aquí tenemos dos opciones, crear una instancia EC2 en amazon web services e instalar Cloud9 en esa instancia o podemos crear nuestro propio servidor en una máquina local, servicios cloud, servicios de hosting, etc. En nuestro servidor instalaremos Cloud9 y se comunicará mediante ssh con los servicios de amazon. Esta última será nuestra opción y sobre la que tratará este post.

Aprenderemos a instalar todas las dependencias necesarias para poder ejecutar Cloud9 en nuestro servidor y que podamos acceder a él mediante la consola Amazon Web Services. Para ello crearemos un Dockerfile donde automatizaremos la instalación de dependencias y se creará una imagen lista para funcionar y que podremos utilizar siempre que usemos el mismo enviroment de Cloud9.

El Dockerfile es el siguiente y a continuación iremos analizando el por qué de las partes que lo forman:
```docker
FROM ubuntu

#Establecemos la zona horaria
ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/Madrid"

#Creamos el usuario cloud9
RUN adduser --disabled-password --gecos '' cloud9

#Actualizamos e instalamos dependencias necesarias
RUN apt-get update -yq  && apt upgrade -yq \
    && apt-get -yq install curl gnupg ca-certificates \
    && curl -L https://deb.nodesource.com/setup_12.x | bash \
    && apt-get update -yq \
    && apt-get install -yq \
        python3 \
        python2 \
        openssh-server \
        build-essential \
        locales-all \
        nodejs 

#Configuramos el servidor ssh con la clave privada proporcionada por amazon
RUN sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config \
    && mkdir /home/cloud9/.ssh \
    && echo '' > /home/cloud9/.ssh/authorized_keys \
    && chmod u=rwx,g=rx,o=rx /home/cloud9 

#Exponemos el puerto 22 para que podamos comunicarnos mediante ssh con la máquina
EXPOSE 22/tcp

#Creamos un script que levante el servidor ssh y un bash para poder interactuar con la máquina, le damos permisos de ejecución
RUN echo '#!/bin/bash\nservice ssh start\n/bin/bash' > /home/cloud9/start.sh
RUN chmod +x /home/cloud9/start.sh

#Configuramos para que al entrar a la máquina lo primero que se ejecute sea nuestro script start.sh
ENTRYPOINT ["/bin/sh", "-c", "./home/cloud9/start.sh"]
```
En primer lugar, he elegido una imagen ubuntu ya que es la que recomienda amazon en su documentación. Después nos encargamos de ajustar la zona horaria, que es necesario para la instalación de node.js (requisito indispensable para que la instancia funcione).

Una vez hecho esto creamos el usuario que utilizará cloud9 para trabajar. Yo elegí el nombre cloud9 por facilidad nemotécnica pero puedes llamarlo como quieras (siempre que al crear el enviroment de cloud9 en la consola de amazon establezcas el mismo nombre).

Luego instalamos todas las dependencias necesarias:
- Python3 y Python2 (python2 no está en la documentación de amazon como requisito indispensable pero al ejecutar el servicio de Cloud9, este nos lo pide por lo que debe quedar instalado).
- openssh-server, esto nos sirve para comunicar nuestra instancia con los servidores de amazon.
- build-essential, nos lo pide amazon para poder tener gcc y make.
- locales-all, este paquete contiene los datos regionales precompilados de todas las regiones que se pueden elegir.
- node.js, muy necesario ya que cloud9 utiliza el motor de node para poder correr en el servidor.

Posteriormente procedemos a configurar el server ssh para establecer la autenticación mediante claves ssh y configurar el archivo authorized_keys para incluir la clave que nos proporciona amazon (linea del echo). Este un paso importante que no he incluido en el Dockerfile (como podreis comprobar paso una cadena vacía al archivo authorized_keys), ya que la clave privada que proporciona amazon en de uso único y privado. Aquí tendreís que crear una instancia de Cloud9 en la Consola de Amazon y en el menu de configuración os aparecerá. Simplemente sustuís la cadena vacía por vuestra clave y os funcionará.

Una vez hecho esto simplemente compilamos la imagen y ya quedaría listo para hacer el run.

He de aclarar que no se ha instalado el servicio de amazon ya que al realizar la conexión ssh con éxito automaticamente se instala Cloud9 en nuestro contenedor.

Ya estaría todo lo que quería comentaros, espero que hayais disfrutado y aprendido un poco más, que es de lo que se trata.

