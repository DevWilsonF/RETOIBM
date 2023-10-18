# Implementación de MQ en un contenedor Docker

## Introduccion
Esta guía proporciona un conjunto de instrucciones paso a paso para crear una imagen con MQ, luego crear un contenedor con esa imagen y crear diferentes colas de mensajes en el MQ del contenedor, incluyendo colas locales y una cola alias. Además, se detallará cómo inyectar un mensaje en una de las colas a través de RFHUtil.

## Retos
- Crear una imagen con MQ
- Crear el contenedor con la imagen del MQ
- En el MQ del contenedor crear 2 colas locales
- En el MQ del contenedor Crear 1 cola alias
- Inyectar un mensaje a una de las colas por medio de RFHUtil

## Requisitos

- Docker
- RFHUtil


## Crear una imagen MQ en Docker

- Creacion Colas locales y alias.
- Cree un archivo llamado [config.mqsc](config.mqsc) que contenga un script para definir y configurar colas y alias de cola. Además,  asigna permisos y autorizaciones al usuario "app".

``` Bash
define ql(DEV.QUEUE.IN) replace
define ql(DEV.QUEUE.OUT) replace
define ql(DEV.QUEUE.ERROR) replace
define qa(DEV.QUEUE.ALIAS) replace

SET AUTHREC PROFILE(*) OBJTYPE(QMGR) PRINCIPAL('app') AUTHADD(ALL)
SET AUTHREC PROFILE(DEV.DEAD.LETTER.QUEUE) OBJTYPE(QUEUE) PRINCIPAL('app') AUTHADD(ALL)
SET AUTHREC PROFILE(DEV.QUEUE.*) OBJTYPE(QUEUE) PRINCIPAL('app') AUTHADD(ALL)
```

- Creaacion de imagen por medio de un Dockerfile.
- En el archivo [Dockerfile](Dockerfile) se indica que se debe copiar el archivo [config.mqsc](config.mqsc), el cual contiene la creación y configuracion de permisos de colas locales y alias.

``` Bash
FROM ibmcom/mq

COPY config.mqsc /etc/mqm/
```

- El siguiente comando crea la imagen en docker.

 ``` Bash
 docker build -t mymq .  
 ```
## Ejecutar el contenedor con la imagen del MQ

El siguiente comando contiene:

- Nombre de contenedor: mqreto2.
- El nombre del gestor de colas: MQACTIVIDAD2.
- Los puertos  donde se va a publicar  1418 y 9443.
- La imagen creada anteriormente: mymq.

``` Bash
docker run --rm --name mqreto2 --env LICENSE=accept  --env MQ_QMGR_NAME=MQACTIVIDAD2 --publish 1418:1414 --publish 9443:9443 --detach mymq 
```


## Inyectar un mensaje a una de las colas por medio de RFHUtil

RFHUtilC es una herramienta útil para aquellos que necesitan trabajar con mensajes en un entorno IBM MQ

- Configure RFHUtilC para establecer la conexion con el gestor de mensajes desplegado en Docker
- Se indica la ruta a la cual se va a conectar, en este caso DEV.ADMIN.SVRCON/TCP/localhost(1418), y el nombre de la cola en la que se va a inyectar el mensaje.

![image](https://media.github.ibm.com/user/420325/files/46a48f36-1d7d-456e-b3ca-b8bcc0a74ccd)

- En la interfaz oprima el boton "Set Conn Id" para configurar los datos de usuario "admin" y contraseña "passw0rd".

![image](https://media.github.ibm.com/user/420325/files/ccb849ba-6f3b-41ae-8e2c-5107c134db74)

- Para esta prueba se utilizara un archivo con un mensaje en formato JSON  en la pestaña "Open File" seleccione el archivo.
- Rfhutil muestra el tamaño de la data

![image](https://media.github.ibm.com/user/420325/files/f726b073-1e9c-4c5e-91aa-7bdfd961b091)

- Oprima el boton "Write Q" el cual enviara el mensaje a la cola indicada. 

<img width="399" alt="image" src="https://media.github.ibm.com/user/420325/files/8d21fb40-e135-4092-8f84-31151d9a66c1">

- Visualice el resultado en la interfaz grafica de MQ, en la URL https://127.0.0.1:9443/ibmmq/console/#/manage/qmgr/MQACTIVIDAD2/queues

![image](https://media.github.ibm.com/user/420325/files/1f9cbb9f-c7a9-47d6-975c-c32fb1524304)









