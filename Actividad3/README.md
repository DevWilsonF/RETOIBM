# Aplicación ACE para conectarse a MQ por medio de políticas

## Introducción
En este proyecto, se propone desarrollar una aplicación con IBM App Connect Enterprise (ACE) que se conecte a IBM MQ y escuche una cola específica para procesar los mensajes entrantes. La aplicación debe ser capaz de responder a los mensajes según su contenido, utilizando políticas para la gestión de la conectividad y la seguridad.

## Retos
Crear una aplicación con ACE que se conecte a MQ y escuche una cola, la conexión debe ser por medio de políticas, el flujo debe comportarse según las siguientes condiciones: 
- Si el mensaje dice hola responda en otra cola con el mensaje "Saludo + HHMMDD"
- Si el mensaje dice otra cosa responda en otra cola con el mensaje "Error"

## Requisitos

- IBM App Connect Enterprise Toolkit
- IBM App Connect Enterprise Console
- Ejecución de contenedor MQ [Implementación de MQ en un contenedor Docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad2).
- Docker
- RFHUtil

# Crear aplicación que se conecte por colas.
- Crear una aplicación con un flujo de mensajes

![image](https://media.github.ibm.com/user/420325/files/dde83298-ec07-4ab8-9fd7-31c16811c051)


- Dentro del flujo establecer los siguientes nodos.
  - Nodo MQInput
  - Nodo Compute
  - Dos Nodos MQOutput(Error, Saludo)
  
  ![image](https://media.github.ibm.com/user/420325/files/7964c531-6710-40dd-a7ab-e9ff25b8bbfd)


- Crear una política para la conexión con el gestor de colas 
- EL gestor de colas al cual se va a conectar la aplicacion  va ser la que se desplegó en [Implementación de MQ en un contenedor Docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad2)

![image](https://media.github.ibm.com/user/420325/files/a1bbc6a7-8255-42b8-b29a-3c393cd82eda)


- Configurar los atributos de la política.
  - Tipo sera MQEndpoint. 
  - Nombre de el gestor de colas al cual se va a conectar.
  - Nombre  del host al cual se va a conectar.
  - Puerto de escucha.
  - El nombre del canal.

![image](https://media.github.ibm.com/user/420325/files/5cc417fb-20d7-4e8c-8635-556eb036bb95)


# Configuración de las propiedades de los nodos.
  ## Nodo MQInput.
  Nombre de la cola a la cual se a enviara el mensaje
  
  ![image](https://media.github.ibm.com/user/420325/files/adb4a09f-1a99-42ce-8709-c631102bcb92)

  
  Formato de salida: JSON
  
  ![image](https://media.github.ibm.com/user/420325/files/61152591-e600-49f3-b387-f2aa4098fb55)

  Establecer conexión por medio de políticas
  
  ![image](https://media.github.ibm.com/user/420325/files/9bf28228-6b00-4470-a2d2-c74f86e9a4e3)
  
  ## Nodo MQOutput (ERROR)
  
  Nombre de la cola de salida que se trasmitirá el mensaje en caso de ERROR
  
  ![image](https://media.github.ibm.com/user/420325/files/e81e34f0-41c3-4d9b-b50b-4c931b86f932)

  
  Establecer conexión por medio de políticas
  
  ![image](https://media.github.ibm.com/user/420325/files/96e917f3-56d3-44e0-ac19-579272bc8bfe)

  
  ## Nodo MQOutput (SALUDO)
  
  Nombre de la cola de salida que se trasmitirá el mensaje en caso de ser exitoso
  
  ![image](https://media.github.ibm.com/user/420325/files/eb231ee4-d882-41d0-9663-ab821b2b52b6)


  
  Establecer conexión por medio de políticas
  
  ![image](https://media.github.ibm.com/user/420325/files/be1d0de1-eb76-4e23-94f5-c053b294b26b)

 
- configurar el archivo ESQL del nodo compute
  - Se declaran variables las cuales van a generar la fecha.

  - Establecer Condiciones
    -  Si el mensaje dice hola responda en otra cola con el mensaje "Saludo + HHMMDD" 
       - El mensaje de salida se trasmitirá por la terminal 1 "Out1" 

    - b. Si el mensaje dice otra cosa responda en otra cola con el mensaje "Error"
      - El mensaje de salida se trasmitirá por la terminal 2 "out2"
  
  ``` Bash
  CREATE COMPUTE MODULE flowsaludo_Compute
	CREATE FUNCTION Main() RETURNS BOOLEAN
	BEGIN
		DECLARE INDATE TIME CURRENT_TIME;
		DECLARE INDATEFORMAT CHARACTER 'hh:mm:ss';
		DECLARE OutMessageDate CHARACTER CAST(INDATE AS CHARACTER FORMAT INDATEFORMAT);
		CREATE LASTCHILD OF OutputRoot DOMAIN('JSON');	
		IF InputRoot.JSON.Data.Saludo = 'Hola' THEN
			  SET OutputRoot.JSON.Data.Salida ='Welcome' ||' '|| OutMessageDate;
			  PROPAGATE TO TERMINAL 1;
		ELSE 
			SET OutputRoot.JSON.Data.Salida = 'ERROR';
			PROPAGATE TO TERMINAL 2;
		END IF;
		RETURN TRUE;
	END;

  END MODULE;
  ```
  

# Pruebas de salida

- Con RFHUtilc se envía un archivo con un mensaje en formato JSON con el mensaje de entrada a la cola DEV.QUEUE.1
- Para ver la configuración de RFHUTIL  [Implementación de MQ en un contenedor Docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad2)

## Si el mensaje dice hola responda en otra cola con el mensaje "Saludo + HHMMDD"
  
- Cargue un documento tipo JSON con el mensaje "Hola"
  
``` Bash
{
  "Saludo":"Hola"
}
```
  
  ![image](https://media.github.ibm.com/user/420325/files/a6335912-e5f1-49d0-b57e-eeb3324082a6)

  ![image](https://media.github.ibm.com/user/420325/files/7d0f6b0d-100a-4a3f-a941-1958f374fb09)
  
- La verificación de el mensaje de respuesta usamos RFHUTILC 
- La salida estará expuesta en la cola DEV.QUEUE.2
  
  ![image](https://media.github.ibm.com/user/420325/files/21cf1cc8-0091-40df-8ed7-bf9fe2986b91)
  
  ![image](https://media.github.ibm.com/user/420325/files/5ce453c1-fb8e-473d-a832-3c2f0cb1c180)
  
  ![image](https://media.github.ibm.com/user/420325/files/cb097f4b-b696-4483-a6ae-e7645da702ec)

    

  
## b. Si el mensaje dice otra cosa responda en otra cola con el mensaje "Error"
  
- Cargue un archivo con el mensaje en formato JSON que contenga un registro diferente a "Hola" para visualizar el error.

``` Bash
{
  "Saludo":"blabla"
}
```   
   ![image](https://media.github.ibm.com/user/420325/files/edc07dcf-0449-46c3-a90a-119f304f0f6f)

   ![image](https://media.github.ibm.com/user/420325/files/4d6a9a37-fd69-40e0-b9fd-9f54a685400d)

   

- La verificación de salida estará expuesta en la cola DEV.QUEUE.3
   
   ![image](https://media.github.ibm.com/user/420325/files/b35cf5a1-11ce-4a47-be24-731f7683ac40)
   
   ![image](https://media.github.ibm.com/user/420325/files/d5fba249-314c-459a-ae17-dbacd0c5ef6d)

   ![image](https://media.github.ibm.com/user/420325/files/f6d03d6e-eb76-4ba3-96aa-5e7fb8a9fd27)
   
   
# Creación imagen y contenedor en docker

## Creación del Dockerfile

- Para poder empaquetar y generar la imagen en Docker, es necesario que se realice la instalación de algunos paquetes de MQ y se configure la conexión con el gestor de mensajes a través de políticas.

El script contenido en los archivos [install-mq-client-prereqs.sh](install-mq-client-prereqs.sh) y [install-mq.sh](install-mq.sh) realiza una serie de tareas necesarias para la instalación y configuración de MQ.

En el archivo [install-mq-client-prereqs.sh](install-mq-client-prereqs.sh), lleva a cabo la instalación de algunos paquetes, verificando previamente si los comandos "microdnf" y "rpm" están disponibles en el sistema. Si los comandos están disponibles, se procede a la instalación de los paquetes adicionales necesarios. Por último, se elimina el caché utilizado por el comando "microdnf".

Por otro lado, el script del archivo [install-mq.sh](install-mq.sh)  se encarga de la descarga e instalación de los componentes necesarios del cliente IBM MQ, utilizando paquetes RPM. Esta tarea es fundamental para que la aplicación pueda conectarse con éxito al gestor de mensajes MQ.



- Configuracion [Dockerfile](dockerfile)


``` Bash

FROM cp.icr.io/cp/appc/ace:12.0.8.0-r1@sha256:432986d77b781291b057ec5a01327cc9417df0d143982e00bc45302674923856

USER root

# Los paquetes MQ a instalar
ARG MQ_URL
ARG MQ_URL_USER
ARG MQ_URL_PASS
ARG MQ_PACKAGES="MQSeriesRuntime*.rpm MQSeriesJava*.rpm MQSeriesJRE*.rpm MQSeriesGSKit*.rpm MQSeriesClient*.rpm"
ARG INSTALL_JRE=0

ARG MQM_UID=888

COPY *.bar /tmp
COPY install-mq.sh /usr/local/bin/
COPY install-mq-client-prereqs.sh /usr/local/bin/

RUN export LICENSE=accept \
  && . /opt/ibm/ace-12/server/bin/mqsiprofile \
  && set -x && for FILE in /tmp/*.bar; do \
     echo "$FILE" >> /tmp/deploys && \
     ibmint package --compile-maps-and-schemas --input-bar-file "$FILE" --output-bar-file /tmp/temp.bar  2>&1 | tee -a /tmp/deploys && \
     ibmint deploy --input-bar-file /tmp/temp.bar --output-work-directory /home/aceuser/ace-server/ 2>&1 | tee -a /tmp/deploys; done \
  && ibmint optimize server --work-dir /home/aceuser/ace-server \
  # Instalar MQ. Para evitar un error de "archivo de texto ocupado" usamos Sleep antes de instalar.
  && chmod -R ugo+rwx /home/aceuser/ \
  && chmod u+x /usr/local/bin/install-*.sh \
  && sleep 1 \
  && install-mq-client-prereqs.sh $MQM_UID \
  && install-mq.sh $MQM_UID \
  && chown -R 1001:root /opt/mqm/*  \
  && chown 1001:root /usr/local/bin/*mq* \
  && mkdir -p /var/mqm/data \
  && chown -R 1001:root /var/mqm \
  && chmod -R 777 /var/mqm
  

ENV MQCERTLABL=aceclient

USER 1001
```

El siguiente comando crea la imagen en docker.

``` bash
docker build -t acereto3 --build-arg MQ_URL=https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqdev/redist/9.3.2.0-IBM-MQC-Redist-LinuxX64.tar.gz --file Dockerfile .
```

Ejecutar el contenedor en el puerto 7601 & 7801 con el siguiente comando.

``` bash
docker run --name acereto3 -d -p 7601:7600 -p 7801:7800 -e LICENSE=accept acereto3                           
```  
  



