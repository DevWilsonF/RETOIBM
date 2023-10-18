# Integración de nodos MQ y consumo de API REST en una aplicación ACE

## Introducción

Se desarrolla una solución que permite obtener información de países a través de una API REST externa y, posteriormente, procesar dicha información en una cola de MQ para su posterior procesamiento. 

## Reto
- Crear una aplicación con ACE que active su flujo por medio de nodos MQ. Posteriormente, por medio de un nodo HTTP consuma el siguiente endpoint:   https://restcountries.com/v3.1/alpha/{id} (API REST – Países) para obtener la información del país que se está consultando.

## Requisitos

- IBM App Connect Enterprise Toolkit
- Ejecución de contenedor MQ [Implementación de MQ en un contenedor Docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad2).
- RFHUTILC
- Docker

##Respuestas esperadas:
Exitosa:

![image](https://media.github.ibm.com/user/420325/files/6e0f572e-c8d6-4bc1-806a-f973ac0c0c60)

Error:

![image](https://media.github.ibm.com/user/420325/files/e53f2088-9d0b-429f-8831-b58e1aaf7e50)


Crear una aplicación en ACE y dentro de esta crear un flujo.

El flujo Contendrá:
- Nodo HTTPRequest
- Nodo MQ Input
- Dos Nodos MQ Output (salida: Exitosa - Error)
- Tres Nodos Compute 

Crear y configurar una política para la conexión con el gestor de colas  de IBM MQ
- EL gestor de colas al cual se va a conectar la aplicación, va ser la que se desplegó en [Implementación de MQ en un contenedor Docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad2).

![image](https://media.github.ibm.com/user/420325/files/0ab079f7-c9d2-47f8-a0d2-79f64f4769a8)

# Configuración de Nodos

## Estructura Mensaje de flujo

![image](https://media.github.ibm.com/user/420325/files/43f9d8f4-9cf2-46c5-9b3c-db4eee3dee4c)

## MQ Input
Indicar la cola en la que se enviara el mensaje de entrada que contendrá el ID del país a consultar en formato JSON, y se configura la conexión mediante políticas.

![image](https://media.github.ibm.com/user/420325/files/1c0125cb-b960-4651-977b-998ca246e311) ![image](https://media.github.ibm.com/user/420325/files/7a5b1e47-e41a-4909-a6a7-b4e98aa66c30) ![image](https://media.github.ibm.com/user/420325/files/ea6c1d73-5804-412d-8485-ee534edbf05d)

## MQ Output Exitoso

Indicar la cola donde va a recibir el mensaje de respueta y se configura la conexion por medio de políticas.

![image](https://media.github.ibm.com/user/420325/files/0f195407-71d2-41ab-aa09-d43213d90b14) ![image](https://media.github.ibm.com/user/420325/files/41ff2df4-41b8-4aeb-8d3d-f248ebcac23a)

## MQ Output Error

Indicar la cola donde va a recibir el mensaje de respueta y se configura la conexion por medio de políticas.

![image](https://media.github.ibm.com/user/420325/files/029febad-d0b9-4cd0-9bcc-acd40662e0c5) ![image](https://media.github.ibm.com/user/420325/files/51028638-9e8f-4784-9120-1f5ae4913f8e)


## Nodo HTTPRequest

El nodo HTTPRequest se conecta de la siguiente manera:
- la salida (OUT) se conecta al nodo successfulcompute1 para generar la respuesta Exitosa esperada, mientras que la salida (ERROR) se conecta al nodo Errorcompute2 para generar la respuesta de Error esperada.



![image](https://media.github.ibm.com/user/420325/files/fe3d8291-85d5-4150-9ff4-e00e26b84024)


Indicar en las propiedades en que url se realiza la petición para traer los datos. la URL es https://restcountries.com/v3.1/alpha/

![image](https://media.github.ibm.com/user/420325/files/1ec6ff14-4352-4a1f-b9bb-94580de4c4f9)

En la pestaña "HTTP settings" indicar el método HTTP en este caso es GET y en la pestaña "Responce Message Parsing" indicar el formato de dominio que va a tener la respuesta en este caso JSON.

![image](https://media.github.ibm.com/user/420325/files/9cd65ec3-16df-46b9-a08a-d7e7bc28d8e7) ![image](https://media.github.ibm.com/user/420325/files/6c1c331d-18c9-471f-8a7c-ba71a28d72ad)


## Nodo Compute 

La función de este nodo es recibir el mensaje de entrada que llega desde el nodo MQ input en formato JSON el cual contiene el ID del país a consultar.
Se establece la URL de solicitud HTTP destino para que la aplicación  pueda consumir  la APIRest  del país. 

Código ESQL
``` Bash
CREATE COMPUTE MODULE flowreto4_Compute
	CREATE FUNCTION Main() RETURNS BOOLEAN
	BEGIN
		SET OutputRoot.Properties = InputRoot.Properties;
		SET Environment.Id = CAST(InputRoot.JSON.Data.Id AS CHARACTER);
		SET OutputLocalEnvironment.Destination.HTTP.RequestURL = 'https://restcountries.com/v3.1/alpha/'||Environment.Id;
		RETURN TRUE;
	END;

END MODULE;
```

## nodo successfulCompute1

La función de este nodo es procesar los datos JSON obtenidos del servicio web REST Estableciendo algunas propiedades de salida transformando los datos de entrada  a una estructura XML de salida.

``` Bash
CREATE COMPUTE MODULE susessfullCompute
	CREATE FUNCTION Main() RETURNS BOOLEAN
	BEGIN
		DECLARE sp1 NAMESPACE 'http://dolab.com/inf/v3.0';
		SET OutputRoot.Properties = InputRoot.Properties;
		SET OutputRoot.XMLNSC.sp1:esbXML.(XML.NamespaceDecl)xmlns:NS1='http://dolab.com/inf/v3.0';
		declare OutXmlTree reference to OutputRoot.XMLNSC.sp1:esbXML;
		SET OutXmlTree.Header.timestamp= CURRENT_TIMESTAMP;
		SET OutXmlTree.Header.responseData.status='exitoso';
		SET OutXmlTree.Body.country.name.common=InputRoot.JSON.Data.Item.name.common;
		SET OutXmlTree.Body.country.currencies.currencie.code = FIELDNAME(InputRoot.JSON.Data.Item.currencies.[1]);
		SET OutXmlTree.Body.country.currencies.currencie.name = InputRoot.JSON.Data.Item.currencies.[1].name;
		SET OutXmlTree.Body.country.currencies.currencie.symbol = InputRoot.JSON.Data.Item.currencies.[1].symbol;
		SET OutXmlTree.Body.country.region= InputRoot.JSON.Data.Item.region;
		SET OutXmlTree.Body.country.subregion= InputRoot.JSON.Data.Item.subregion;
		SET OutXmlTree.Body.country.languages.languaje.name=  InputRoot.JSON.Data.Item.languages.[1];		
		RETURN TRUE;
	END;

END MODULE;
```

## ErrorCompute2

La función de este nodo consiste en generar una respuesta en formato XML en caso de que ocurra un error durante el procesamiento de la petición HTTP, indicando que el país consultado no pudo ser encontrado.

``` Bash

CREATE COMPUTE MODULE ErrorCompute2
	CREATE FUNCTION Main() RETURNS BOOLEAN
	BEGIN
		DECLARE sp1 NAMESPACE 'http://dolab.com/inf/v3.0';
		SET OutputRoot.Properties = InputRoot.Properties;
		SET OutputRoot.XMLNSC.sp1:esbXML.(XML.NamespaceDecl)xmlns:NS1='http://dolab.com/inf/v3.0';
		declare OutXmlTree reference to OutputRoot.XMLNSC.sp1:esbXML;
		SET OutXmlTree.Header.timestamp= CURRENT_TIMESTAMP;
		SET OutXmlTree.Header.responseData.status='error';
		SET OutXmlTree.Body.codigo=InputRoot.HTTPResponseHeader.[2];
		SET OutXmlTree.Body.descripcion='El pais '||Environment.Id||' no se encontro';
		RETURN TRUE;
	END;

	
END MODULE;

```                       


# Creacion imagen y contenedo en docker

[Configuracion docker](https://github.ibm.com/Fabian-Cruz/Retos2.0/tree/main/Actividad3#creaci%C3%B3n-imagen-y-contenedor-en-docker)

El siguiente comando crea la imagen en docker
``` Bash
docker build -t aceretofinal --build-arg MQ_URL=https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqdev/redist/9.3.2.0-IBM-MQC-Redist-LinuxX64.tar.gz --file Dockerfile .
```
El siguiente comando ejecuta el contenedor apartir de la imagen creada
``` Bash
docker run --name acereto4 -d -p 7603:7600 -p 7803:7800 -e LICENSE=accept aceretofinal 
```
