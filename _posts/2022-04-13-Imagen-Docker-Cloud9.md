---
title: Crear contenedor para implementar el IDE de Amazon, Cloud9, en nustro propio servidor
published: true
---
Amazon nos provee de un IDE en la nube muy completo llamado Cloud9. Con este IDE en la nube podremos trabajar codo a codo con nuestros compañeros de manera remota debido a que cada cambio que haga nuestro compañero lo veremos al instante lo que hace que sea un IDE pensado para el teletrabajo. Cloud9 es capaz de compilar y ejecutar nuestros proyectos en la nube y es compatible con numerosos lenguajes, entre los que se encuentran C, C ++, PHP, Ruby, Perl, Python, JavaScript con Node. js y Go.

Para poder acceder a este entorno de desarrollo, debemos tener una cuenta creada en Amazon Web Services. Una vez tenemos la cuenta creada, iremos al enlace de cloud9 <https://console.aws.amazon.com/cloud9/>.

Aquí tenemos dos opciones, crear una instancia EC2 en amazon web services e instalar Cloud9 en esa instancia o podemos crear nuestro propio servidor en una máquina local, servicios cloud, servicios de hosting, etc. En nuestro servidor instalaremos Cloud9 y se comunicará mediante ssh con los servicios de amazon. Esta última será nuestra opción y sobre la que tratará este post.

Aprenderemos a instalar todas las dependencias necesarias para poder ejecutar Cloud9 en nuestro servidor y que podamos acceder a él mediante la consola Amazon Web Services. Posteriormente crearemos un Dockerfile donde automatizaremos la instalación de dependencias y se creará una imagen lista para funcionar y que podremos utilizar siempre que usemos el mismo enviroment de Cloud9.

### Inacabado