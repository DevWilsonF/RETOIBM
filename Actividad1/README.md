# Aplicacion ¡Hola mundo! con ACE

## Introduccion

La siguiente documentación proporciona un recorrido por el proceso de desarrollo de una aplicación que genera un mensaje de respuesta en formato JSON con el contenido de "¡Hola Mundo!". En esta documentación, se detalla cómo se crea el flujo de mensajes, así como también se explica cómo desplegar la aplicación en un servidor local y mediante el uso de Docker. Se incluyen instrucciones detalladas sobre cómo configurar el entorno de desarrollo. Además, se proporcionan detalles técnicos sobre las tecnologías utilizadas en la implementación de la aplicación

## Requisitos

- IBM App Connect Enterprise Toolkit
- IBM App Connect Enterprise Console
- Docker

## Retos

- [Crear con ACE una aplicación que emita el mensaje ¡Hola Mundo!](#crear-con-ace-una-aplicación-que-emita-el-mensaje-hola-mundo)
- [Crear una imagen con ACE donde se despliegue un barfile (Creado a partir del punto 1)](#crear-una-imagen-con-ace-donde-se-despliegue-un-barfile-creado-a-partir-del-punto-1)



## Crear con ACE una aplicación que emita el mensaje ¡Hola Mundo!

- En App Connect Enterprise toolkit, Para crear la aplicación, se debe ir a la pestaña de "Nuevo" dentro de la herramienta, seleccionar "Aplicación" y asignarle un nombre descriptivo. 
  - Una vez creada la aplicación, se puede agregar un flujo de mensajes seleccionando "Nuevo" con un clic derecho en el panel del proyecto, y luego seleccionando "Flujo de mensajes" en las opciones.
  -  Se debe asignar un nombre identificador al flujo de mensajes y hacer clic en "Finalizar" para completar la creación del flujo de mensajes.

<img width="400" alt="Captura de pantalla 2023-04-12 181242" src="https://media.github.ibm.com/user/420325/files/bd1aea45-574a-4359-9f63-ef56be121751">

- Dentro del flujo, se agregan los nodos HTTPInput, HTTPReply y un Nodo Compute.
- Se conectan las salidas con las entradas correspondientes en el flujo de la aplicación.

<img width="411" alt="Captura de pantalla 2023-04-12 181503" src="https://media.github.ibm.com/user/420325/files/93e00e42-4164-4fad-8da5-d3badde8b420">

- En la pestaña "basic" del Nodo HTTPInput, se configura el punto final donde se visualizará el mensaje en el flujo de la aplicación.

<img width="450" alt="Captura de pantalla 2023-04-12 184343" src="https://media.github.ibm.com/user/420325/files/d04d39cc-6ca7-40c8-9d36-9100e5df5179">

- El nodo Compute se configura mediante un archivo ESQL, el cual procesará una salida con el mensaje en formato JSON en el flujo de la aplicación.

<img width="254" alt="Captura de pantalla 2023-04-12 184613" src="https://media.github.ibm.com/user/420325/files/89ccf44e-2146-4e09-8ae2-3257b44d7aac">

## Crear el servidor de integración local en el cual se ejecutará la aplicación.
- El siguiente comando se ejecuta en App Connect Enterprise Console:


``` Bash
IntegrationServer --name server1 --work-dir c:\aceserver
```

Se mostrará la confirmación de que el servidor ha sido lanzado con éxito en la App Connect Enterprise Console.

<img width="555" alt="image" src="https://media.github.ibm.com/user/420325/files/5b05090c-8041-4bc1-8d29-168cc63c2a65">

- La aplicación se despliega en el servidor creado para que pueda ser ejecutada correctamente.

<img width="283" alt="image" src="https://media.github.ibm.com/user/420325/files/a1df61ff-6af0-4171-a72b-c89ef927e10a"> <img width="295" alt="Captura de pantalla 2023-04-12 194924" src="https://media.github.ibm.com/user/420325/files/c4c15997-201a-4b16-a4a1-1801be71c24f"> <img width="218" alt="Captura de pantalla 2023-04-12 195011" src="https://media.github.ibm.com/user/420325/files/f449a2d7-c3fd-4c3d-a99b-eab29e1f3eb5">


- Es posible verificar que la aplicación se ha desplegado correctamente en la URL http://localhost:7600 mediante un navegador web.

<img width="680" alt="Captura de pantalla 2023-04-12 203122" src="https://media.github.ibm.com/user/420325/files/2e296bdb-d0a7-4f0f-af56-05a5cdf88b5b">

- Para visualizar el mensaje emitido en la aplicación, se utiliza la URL con el endpoint configurado previamente, en el puerto 7800: http://localhost:7800/MensajeSaludo.

<img width="250" alt="Captura de pantalla 2023-04-12 203441" src="https://media.github.ibm.com/user/420325/files/e047110e-2283-4895-b4ce-7eb6e5006c45">


## Crear imagen con ACE donde se despliegue un barfile (Creado a partir del punto 1)

- Para conocer la ubicación del archivo .bar que se utilizó en el despliegue de la aplicación en el servidor, se puede acceder a las propiedades del archivo y se puede visualizar la ruta donde se encuentra el archivo.

![Captura de pantalla 2023-04-13 140012](https://media.github.ibm.com/user/420325/files/141bb47a-d34e-43b9-8591-09f0e73b82e8)


- Se genera un archivo [Dockerfile](Dockerfile)  para la aplicación.

En el archivo Dockerfile se indica la imagen de ACE que se va a consumir y se desplegará el archivo .bar.
Se copian todos los archivos que terminen en .bar en la carpeta /tmp, luego se procesan, compilan y desempaquetan cada archivo dentro del directorio de trabajo del servidor.

``` Bash
FROM cp.icr.io/cp/appc/ace:12.0.8.0-r1@sha256:432986d77b781291b057ec5a01327cc9417df0d143982e00bc45302674923856

USER root
COPY *.bar /tmp
RUN export LICENSE=accept \
    && . /opt/ibm/ace-12/server/bin/mqsiprofile \
     && set -x && for FILE in /tmp/*.bar; do \
        echo "$FILE" >> /tmp/deploys && \
        ibmint package --compile-maps-and-schemas --input-bar-file "$FILE" --output-bar-file /tmp/temp.bar  2>&1 | tee -a /tmp/deploys && \
        ibmint deploy --input-bar-file /tmp/temp.bar --output-work-directory /home/aceuser/ace-server/ 2>&1 | tee -a /tmp/deploys; done \
    && ibmint optimize server --work-dir /home/aceuser/ace-server \
      && chmod -R ugo+rwx /home/aceuser/

USER 1001
```

- El directorio de trabajo debe contener el archivo .bar que se utilizará para el despliegue de la aplicación en el servidor. [.bar](retoUnoSaludoproject.generated.bar) y el [Dockerfile](Dockerfile) 


![Captura de pantalla 2023-04-13 141212](https://media.github.ibm.com/user/420325/files/0c75ceee-11a0-4d3e-9a81-a41394d604d9)


- Se construye la imagen mediante el siguiente comando de Docker, asegurándose de que el directorio de trabajo contenga el archivo .bar:


``` Bash
docker build -t aceappreto1  --file Dockerfile .
```

![Captura de pantalla 2023-04-13 140618](https://media.github.ibm.com/user/420325/files/1c9b1544-274c-4573-bc16-4c1d6a6d97e0)

- Para correr el contenedor de la imagen creada, se utiliza el siguiente comando:

``` Bash
docker run -d --name aceappreto1 -p 7601:7600 -p 7801:7800 -e LICENSE=accept aceappreto1
```

![Captura de pantalla 2023-04-13 141551](https://media.github.ibm.com/user/420325/files/b6914177-d81c-4303-a93d-47369d345d0b)



- En la interfaz web de App Connect Enterprise, podra visualizar si la aplicacion esta desplegada y en ejecucion http://localhost:7601/ 

![Captura de pantalla 2023-04-13 141824](https://media.github.ibm.com/user/420325/files/3b994d1d-1dd9-407b-8132-7f9df094e2c8)


- comprobar que el mensaje se esté visualizando correctamente en el endpoint configurado.  http://localhost:7801/MensajeSaludo

![Captura de pantalla 2023-04-13 142145](https://media.github.ibm.com/user/420325/files/134d8ca1-9228-47fc-ad5b-eb404612f546)





