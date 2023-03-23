# Manual Docker
En este manual se cubrirán las partes basicas para el uso y manejo de contenedores e imagenes en Docker

## Indice
 - [Comandos Esenciales para Docker](#comandos-esenciales-parap-docker)
 - [Comandos basicos para contenedores](#comandos-basicos-de-docker)
 - [Uso de Dockerfile](#uso-de-dockerfiles)
 - [Uso de Docker Registry](#uso-de-docker-registry)
 - [Uso de Docker Compose](#uso-de-docker-compose)

## Comandos Esenciales para Docker
* Obervar los contenedores actualmente activos <br/> `docker ps` <br/>
* Eliminiar contenedores que no estan activos <br/> `docker rm id_contenedor o nombre_contenedor` <br/>
* Eliminar imagenes <br/> `docker rmi id_imagen o nombre_imagen:version` <br/>
* Observar las imagenes guardadas localmente <br/> `docker images` <br/>
* Detiene contenedores activos <br/> `docker stop id_contenedor o nombre_contenedor` <br/>

![diag_docker_containers](imagenes/diag_docker_containers.png)

[Regresar al inicio](#indice)

## Comandos basicos de Docker
### Crear un contenedor
Para poder crear un contenedor primero es necesario saber sobre que sistema estara trabajando dicho contenedor. En este caso haremos uno sobre centos 7

El comando para hacer lo anterior es el siguiente:

```
docker create --name centosPersonal -itp 7500:80 centos:7
```

Lo que ocurre con el comando anterior es lo sugiente:
* Con docker create decimos que vamos a crear un contenedor
* La bandera --name sirve para darle un nombre al contenedor que crearemos(en este caso CentosPersonal)
* La bandera -p nos ayuda a determinar que puerto será el utilizado por la computadora(en este caso el 7500) y que puerto utilizara el contenedor(en este caso el 80)
* La bandera -i habilita la entrada de datos estandar
* La bandera -t habilita una terminal en el contenedor
* El centos:7 nos sirve para determinar que imagen de base será utilizada en el contenedor en este caso sera un centos cuya version es la 7
* *Si la imagen deseada no se encontrara localmente, automaticamente buscara una en dockerhub*

![creacion_imagen](imagenes/creacion_imagen.png)

Habiendo hecho esto podemos confirmar que la version que se instalo de imagen es la de centos7 con ayuda del siguiente comando
```
docker images
```
*Con este comando listamos todas las imagenes instaladas localmente*

![imagenes](imagenes/imagen.png)

Para revisar que nuestro contenedor se haya creado exitosamente haremos uso del siguiente comando:
```
docker ps -a
```
*El comando docker ps nos ayuda a saber que contenedor esta corriendo y con la bandera -a mostramos incluso aquellos que no*

![contenedores](imagenes/contenedor.png)

**Listo, has creado tu primer contenedor en Docker :D!**

As Abraham Lincoln once said:
> *Pardon my French!*

<br />
### Entrar a un contenedor

Para poder correr el contenedor que hemos creado hacemos uso del siguiente comando:
```
docker start centosPersonal
```
![correr_contenedor](imagenes/start_container.png)

Para verificar que nuestro contenedor este corriendo hacemos uso del siguiente comando:
```
docker ps
```
![verificar_correr_contenedor](imagenes/check_running_container.png)

Para entrar a nuestro contenedor para instalar alguna aplicacion o cambiar alguna configuracion hacemos uso del siguiente comando:

```
docker start -ai centosPersonal
```
* La bandera -a habilita la salida estandar y la salida de errores para el contenedor
* La bandera -i habilita la entrada estandar para el contenedor

![entrar_contenedor](imagenes/enter_container.png)
<br /><br />
### Crear un servidor

Una vez dentro de nuestro contenedor podemos instalar casi cualquier cosa.<br />
En este ejemplo intalaremos apache, para completar este proposito haremos uso del siguiente comando:
```
yum install httpd
```
![instalar_apache](imagenes/install_apache.png)

Ahora lo que haremos sera modificar la pagina que se mostrara cuando hechemos a andar el contenedor.<br />
Para poder completar este objetivo haremos uso de vi

![cambiar_index](imagenes/modificar_index.png)

Adentro del index que acabamos de crear pondremos un saludo al mundo porque es importante siempre hacerlo.

![hola_mundo](imagenes/saludo_desde_contenedor.png)

Para hacer cambios hacemos uso del `i` y para guardar cambios hacemos uso del `:w` y para salirnos de vi es con el comando `:q`

Cuando hayamos finalizado con todo lo anterior nos saldremos del contenedor con ayuda del siguiente comando:
```
exit
```
![salir_contenedor](imagenes/exit_container.png)

En este punto es importante notar que los cambios que hemos hecho se queda en nuestro contenedor, pero la imagen base que hemos utilizado sigue sin cambiar.<br />
Esto quiere decir que si hacemos otro contenedor a partir de la imagen de centos7 que tenemos, el contenedor no tendra el apache instalado.<br />
Para evitar que nuestros cambios sean perdidos haremos uso del siguiente comando:
```
docker commit id_contenedor nombre_imagen
```
![crear_imagen_nueva](imagenes/imagen_centos_apache.png)
* El el campo de id_contenedor se debe poner el id del contenedor que queremos guardar sus cambios
* En el campo de nombre_imagen se pondrá el nombre de la nueva imagen a generar o se puede sobreescribir alguna otra imagen

Confirmamos la creacion de una nueva imagen con el siguiente comando:
```
docker images
```
![nueva_imagen_centosapache](imagenes/nueva_imagen_centosapache.png)

Ya que tenemos todo preparado, lo unico restante es observar si funciona o no nuestro pequeño servidor. <br />
Esto lo haremos con ayuda del siguiente comando
```
docker run -p 7500:80 centos_apache:v1 /usr/sbin/httpd -D FOREGROUND
```
![comando_correr_contenedor](imagenes/probar_servidor.png)
* Con el comando */usr/sbin/httpd -D FOREGROUND* nos aseguramos que cuando el contenedor corra tambien se heche a andar el servidor apache. <br/>
**Este comando causará que nuestra terminal se quede en estado de stand by**

En nuestro navegador preferido nos conectamos al servidor apache, como esta en un contenedor se veria asi su ip y el resultado de la peticion

![resultado](imagenes/checar_servidor.png)

**_Puuummm!!!!! Ya creaste tu propio servidor dentro de un contenedor :D!!!_**


Si no quedo claro como se hace el uso de puertos, en el siguiente diagrama encuentra la conexion del ejercicio anterior

![diag_conexion_puertos](imagenes/diag_conexion_puertos.png)

[Regresar al inicio](#indice)

## Uso de DockerFiles

Si nos dimos cuenta, es bastante complicado el quere levantar un servidor con docker, son bastantes comandos tanto para la creacion del propio contendor como para el montaje y funcionamiento de docker.

Para ahorrarnos esto existe una tecnica milenaria llamada Dockerfile, esta tecnica nos permite en un solo paso crear un contenedor con caracteristicas y programas ya instalados.

Algunas cosas basicas de como dictaminar el como se comportarar el dockerfile es a traves de los siguientes comandos:

* ADD - Copia los archivos de un lugar en el host adentro del sistema de archivos del contenedor
* CMD - Puede ser usado para ejecutar un comando especifico dentro del contenedor
* ENTRYPOINT -  Selecciona una aplicacion por default para ser utilizada cada vez que un contenedor es creado con la imagen
* ENV - Coloca las variables de entorno para ser utilizadas en el proceso de creado del contenedor
* EXPOSE - Asocia un puerto especifico para habilitar la conexion entre el contenedor y el mundo exterior
* FROM - Defina que imagen base sera utilizada para inciar el proceso de contruccion
* MAINTAINER - Define quien fue el creeador de la imagen
* RUN - Es la directiva central de ejecucion para DockerFiles
* USER - Coloca el UID o nombre de usuario que correra el contenedor
* VOLUME - Es usado para habilitar el acceso de un contenedor a un directorio en la maquina host
* WORKDIR - Seleciona el path donde el comando, definido por el CMD, se ejecutara
* LABEL - Permite añadir una etiqueta a tu imagen

Exempli gratia haremos el mismo ejemplo de como montar un servidor apache en un centos 7.

Primero debemos crear la carpeta de nuestro contenedor
```
mkdir ~/exemplo
```
![crear_carpeta](imagenes/carpeta_exemplo.png)

Nos vamos a cambiar a la carpeta de nuestro proyecto
```
cd ~/exemplo
```
![cambiar_carpeta](imagenes/cambiarse_exemplo.png)

Creamos un documento llamado Dockerfile
```
touch Dockerfile
```
![crear_dockerfile](imagenes/crear_dockerfile.png)

Vamos a configurar nuestro dockerfile con ayuda de vi
```
Vi Dockerfile
```
![modificar_dockerfile](imagenes/comandos_dockerfile.png)

Construimos nuestra imagen con el comando `build`
```
docker build -t "Exemplo:1" .
```
![construir_dockerfile](imagenes/build_image.png)

Hechamos a andar un nuevo contenedor a partir de la imagen que creamos
```
docker run -dp "8500:80" exemplo:1
```
![correr_contenedor_imagen_nueva](imagenes/run_new_image.png)

Comprobamos que se haya creado nuestro nuevo contenedor con el siguiente comando:
```
docker ps -a
```
![comprobar_imagen](imagenes/comprobar_new_image_run.png)

Entramos al contenedor creado
```
docker exec -it id_contenedor bash
```
![entrar_contenedor_imgen_nueva](imagenes/enter_container_new_image.png)

Comprobamos que nuestro servidor se haya creado correctamente al entrar en la siguiente pagina web
```
direccion_ip_host:puerto_contenedor
```
![comprobar_server](imagenes/nuevo_server_online.png)

[Regresar al inicio](#indice)

## Uso de Docker Registry
Docker registry es una manera sencilla de mantener a nuestro alcance las imagenes que nosostros creemos y modifiquemos desde cualquier computadora con acceso a la red. De igual manera estas imagenes pueden ser protegidas por medio de un usuario y contraseña.

Una manera de levantar nuestro propio servicio de Docker Registry es mediante la siguiente linea
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
**El registro solo funciona si es el puerto 5000 del contenedor**

Con el comando anterior lo que hacemos es jalar una imagen de tipo registry y la guardamos como version 2 localmente, asi como el puerto por el cual trabajará que sera el 5000 y siempre estara activado el contenedor <br/>

Por default cualquier Registry con HTTP no es validado como seguro, para poder remediar esto y nos acepte un Registry con HTTP hay que modificar el archivo JSON localizado en: <br/>
- Linux = /etc/docker/daemon.json <br/>
- Mac y Windows = icono de docker -> click derecho -> preferencias -> +Daemon <br/>
- Windows Server = C:\ProgramData\docker\config\daemon.json <br/>

**Si el archivo no existe hay que crearlo**
```
{
  "insecure-registries" : ["myregistrydomain.com:5000"]
}
```
![daemon_docker](imagenes/daemon_docker.png)

Reiniciamos docker para que se guarden los cambios
```
service docker restart
```
![service_restart_docker](imagenes/service_restart_docker.png)

Para probar como se maneja el Docker Registry descargaremos la imagen de hello-world con el siguiente comando
```
docker pull hello-world
```
![pull_hello_world](imagenes/pull_hello_world.png)

Creamos una imagen adicional para que docker interprete que va a pertencer a nuestro registry cuando hagamos push
```
docker tag hello-world ip_address:5000/hola-mundo
```
![docker_tag_mundo](imagenes/docker_tag_mundo.png)

Hacemos push de la imagen hacia nuestro registro
```
docker push ip_address:5000/hola-mundo
```
![docker_push_mundo](imagenes/docker_push_mundo.png)

Quitamos las imagenes locales de hello world para liberar espacio
```
docker image remove hello-world
docker image remove ip_address:5000/hola-mundo
```
![docker_image_remove_1](imagenes/docker_image_remove_1.png)
![docker_image_remove_2](imagenes/docker_image_remove_2.png)

Probamos jalar la imagen de hello-world desde nuestro registro
```
docker pull localhost:5000/hola-mundo
```
![docker_pull_mundo](imagenes/docker_pull_mundo.png)

Como se pudo observar la imagen hola-mundo es la misma que hello-world, la unica diferencia es que hello-world Docker la descarga desde Dockerhub y hola-mundo ya esta en el repositorio local, así se puede trabajar sin conexion a internet con la imagen hola-mundo

### Seguridad en Docker Registry

Crear el usuario y contraseña del Registry
```
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn user passwd > auth/htpasswd
```
![create_pass](imagenes/create_pass.png)

Iniciar un Docker Registry con Seguridad
```
 docker run -d \
  -p 5000:5000\
  --restart=always \
  --name registryMachicao \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```
![Registry_Passwd](imagenes/docker_run_pass.png)

Mientras no estemos logeados en nuestro dominio/registro no podremos hacer push ni pull del mismo y nos mostrara el siguiente mensaje de error
![Not_Log_In](imagenes/not_logged_in.png)

Para poder hacer push y pull nos logueamos con el siguiente comando
```
docker login ip_address:5000
```
![Log_In](imagenes/registry_login.png)

Comprobamos que ya una vez logueados podemos hacer push hacia nuestro Registry
```
docker push ip_address:5000/nombre_imagen
```
![push_login](imagenes/push_login.png)

#### Multiples registros

Creacion de un segundo registro con seguridad en el puerto 5001
```
 docker run -d \
  -p 5001:5000\
  --restart=always \
  --name registry1 \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```
Cabe destacar que habra que poner las direcciones ip de nuestros registros en el daemon.json y reiniciar docker, de igual manera las contraseñas que estan en auth serán compartidas para todos los registros.
<br/><br/>
**Exempli Gratia**<br />
Ejemplo login con user al registro en el puerto 5000
![Log_In](imagenes/registry_login.png)

Ejemplo login con user al registro 5001
![registry1_login](imagenes/registry1_login.png)

Si se requiere hacer uso de un usuario diferente para otro registro se tiene que generar otra carpeta y otro archivo de autenticacion con el usuario deseado
```
mkdir auth1
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth1/htpasswd
```
![create_pass_2](imagenes/create_pass_2.png)

Despues levantamos otro registro con la carpeta diferente
```
 docker run -d \
  -p 5002:5000\
  --restart=always \
  --name registry2 \
  -v "$(pwd)"/auth1:/auth1 \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth1/htpasswd \
  registry:2
```
![registry_2](imagenes/registry_2.png)

Intetaremos acceder al registro en el puerto 5002 con las credenciales de user
![login_denegado](imagenes/login_denegado.png)

Como se pudo observar no son reconocidas en el registro nuevo. <br/>
Ahora intentaremos lo mismo pero con las nuevas credenciales.
![login_aceptado](imagenes/login_aceptado.png)

Como se podrá observar de esta manera podemos tener multiples registros y cada uno de ellos para un usuario diferente.

#### Elminacion de registros

Para eliminar nuestros registro tenemos que ejecutar la siguiente linea de ocidgo
```
docker container stop registry && docker container rm -v registry
```

#### Registros con CI de Gitlab

Para poder hacer uso de docker registry con gitlab es necesario seguir los siguientes pasos
![registry_creation](imagenes/gitlab_registry.PNG)

A partir de este punto nosotros podemos meter nuestras imagenes al registry y llamarlas desde los dockerfiles de gitlab en CI
![call_to_arms](imagenes/llamada_registry.PNG)

[Regresar al inicio](#indice)

## Uso de Docker Compose
Como hemos visto docker nos da varias herramienta# En Construccions poderosas a la hora de querer utilizar contenedores. <br/>
Una herramienta mas que nos da es Compose, esta herramienta nos es util a la hora de querer levantar multiples contenedores interconectados entre si. <br/>
Como tal en un docker compose de declaran servicios(contenedores) y que recursos utilizarán los mismos. De igual manera se configuran las diversas opciones de versiones y comandos necesarios para interconectividad entre los servicios.

El archivo necesario para poder hacer uso de Docker Compose se llama __docker-compose.yml__ <br/>

### Elementos de Docker Compose
- Services <br/>
Dentro de la etiqueta se declaran los contenedores que tendrá el archivo Compose <br/>
![services](imagenes/services.png)

- Images <br/>
Sirve para defninir la imagen a usar para construir el contenedor, puede ser llamada desde DockerHub, localmente o utilizando un DockerFile <br/>
![images](imagenes/images.png)

- Ports
  - Expose
  Sirve para mostrar los puertos para la comunicacion entre contenedores. __No para el sistema__ <br/>
  ![expose](imagenes/expose.png) 
  
  - Port
  Sirve para mostrar los puertos para comunicar el contenedor con el sistema <br/>
  ![ports](imagenes/port.png)
  
- Commands <br/>
Son usados para ejectuar acciones una vez hechado a andar el contenedor <br/>
![commands](imagenes/command.png)

- Volumes <br/>
Son usados para la persistencia de datos generados y usados por los contenedores, de igual manera proporciona un espacio en comun para la comparticion de datos entre contenedores <br/>
  - Normales
  Simplemente se epecifica una locacion y se le deja a Docker que cree el volumen <br/>
  ![volumen_normal](imagenes/volumen_normal.png)
  - Path
  Se define un luagar en el sistema host y se mapea a un lugar en el contenedor <br/>
  ![volumen_path](imagenes/volumen_path.png)
  - Nombrados
  Se define un nombre a utilizar seguido de un lugar, este tipo de volumen es util para compartir datos entre contenedores <br/>
  ![volumen_nombrado](imagenes/volumen_nombrado.png)

- Environment <br/>
Sirve para declarar variables de entorno para la configuracion de aplicaciones de los contendores o para variables que cambien de valor dinamicamente <br/>
![environ](imagenes/environ.png)

- Dependencies <br/>
Sirve para declarar si un servicio necesta que otro ya este activo antes de iniciar a ejecutarse <br/>
![dependencies](imagenes/depen.png)

- Linking <br/>
Sirve para generar aliases para que los servicios puedan comunicarse con otros servicios <br/>
![linking](imagenes/linking.png)

### Exempli Gratia
En este ejemplo haremos una aplicacion de Wordpress con su backend necesario.
Generalmente si quiseramoms hacer lo anterior tomaria algo de tiempo configurar las base de datos asi como el proceso de montado de la misma, de igual manera al instalar Wordpress configurarlo para que los puertos conincidan y se puedan comunicar entre ellos
<br/><br/>
Docker compose nos facilita todo este imbroglio burocratico al poder levantar servicios y conectarlos en un solo paso. <br/>

En el siguiente archivo el cual se llama docker-compose.yml hace todo lo anterior.

```
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
       - wp-content:/var/www/html/wp-content
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
       
volumes:
    db_data: {}
    wp-content: {}
```

En el ejemplo anterior ocurre lo siguiente:
* Se declaran los serivcios de db y wordpress
* En el servicio de db se declara lo siguiente: 
    * La imagen a usar sera de mysql en su version 5.7 
    * Se declara que siempre estara activado el servicio 
    * Se declaran las variables de entorno para conectarse con wordpress
* En el servicio de wordpress ocurre lo siguiente:
    * Se declara que primero debe de estar activo el servicio de db para que se pueda iniciar el servicio de wordpress
    * Se declara que el puerto a utilizar sera el 8000 y dentro del contenedor sera el 80
    * Se declara que siempre estara activo el servicio
    * Se declaran variables de entorno para poder conectarse con la db
* Se declara un volumen llamado db_data el cual sera utilizado por la db para la persistencia de datos
* Se declara el columen wp-content donde seran guardados el contenido de los usuarios de wordpress
 

Para poder construir este proyecto se hace uso del siguiente comando
```
docker-compose up -d
```
#### Diagrama de conexiones

![docker_compose_conns](imagenes/docker_compose_conns.png)

[Regresar al inicio](#indice)