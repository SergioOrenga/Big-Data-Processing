#Descripción de la práctica del módulo de Big Data Processing
*Alumno: Sergio Orenga Roglá*

En el ejercicio de la práctica del módulo de Big Data Processing existen cuatro partes diferenciadas:


1. Fuente de datos en tiempo real que envía información sobre 5 antenas de telefonía y 20 dispositivos móviles. Dicha información es enviada al sistema de mensajes de Apache Kafka, y es generada mediante un simulador ejecutado en un contenedor de Docker en local. 

	**INSERTAR IMAGEN DOCKER**

2. Servidor de Apache Kafka, que se ejecuta en una instancia de máquina virtual en Google Compute Engine de la plataforma Google Cloud Computing. En este servidor se reciben los mensajes en tiempo real desde las fuentes de datos. Una vez configurada la máquina virtual se tienen que establecer las reglas de firewall para que se pueda acceder a ella desde el ordenador local, donde se ejecuta el job de Spark Structured Streaming. Se accede a la máquina virtual de Apache Kafka mediante la consola de Google Cloud Compute por SSH y se configura la instancia, se crea el topic “devices” que es donde se recibirán los datos de entrada, y se ejecuta el consumidor. 

	**INSERTAR 3 IMÁGENES VM KAFKA**

3. Base de datos de PostgreSQL creada en Cloud SQL de Google Cloud Platform. En esta base de datos se crean cuatro tablas desde el job JdbcProvisioner.scala.
	- *`user_metadata`*. En el job indicado anteriormente se realiza una carga inicial de los datos de esta tabla. 
	- *`bytes`*. Esta tabla se rellena en tiempo real desde el job AntennaStreamingJob.scala con el resultado de las métricas agregadas cada 5 minutos:
		- Total de bytes recibidos por antena.
		- Total de bytes transmitidos por id de usuario.
		- Total de bytes transmitidos por aplicación.
	- *`bytes_hourly`*. Esta tabla se rellena en batch desde el job AntennaBatchJob.scala con el resultado de las métricas para un año, mes, día y hora determinados:
		- Total de bytes recibidos por antena.
		- Total de bytes transmitidos por email de usuario.
		- Total de bytes transmitidos por aplicación.
	- *`user_quota_limit`*. Esta tabla se rellena en batch desde el job AntennaBatchJob.scala con el resultado de la métrica para un año, mes, día y hora determinados:
		- Email de usuarios que han sobrepasado la cuota por hora.
**INSERTAR 5 IMÁGENES SQL**
4. Job de Spark Structured Streaming que realiza las operaciones para obtener las métricas propuestas y almacenar los resultados en las tablas de la base de datos de PostgreSQL, y también almacena y lee los datos en formato PARQUET en una carpeta local. En el archivo comprimido del proyecto se encuentra todo el código desarrollado, el cual está comentado con la explicación de las tareas desarrolladas, junto con los archivos en formato PARQUET generados. Consta de tres partes:
	- *Provisioner*. Se encarga de crear las tablas en PostgreSQL, así como de cargar los datos iniciales de la tabla user_metadata (JdbcProvisioner.scala).
	- *Speed Layer*. Procesa los datos que le llegan desde el servidor de Apache Kafka en tiempo real. Se compone de:

		- Interfaz StreamingJob.scala. La cual tiene un método “run”, en el que se definen todos los argumentos de entrada que se tienen que introducir para ejecutar el job, y además encadena todos los métodos junto con los parámetros que se les pasan para que tengan sentido y se ejecuten correctamente.

		- Implementación de la interfaz AntennaStreamingJob.scala, que extiende la interfaz StreamingJob.scala. Aquí se desarrollan los métodos definidos en la interfaz. Los argumentos de entrada que se le pasan son los siguientes:
			- IP externa del servidor de Apache Kafka con el puerto 9092
			- El nombre del topic de los datos de Kafka para procesar los datos de sólo ese topic
			- La URI de conexión al servidor de PostgreSQL en Google Cloud
			- La tabla `user_metadata`
			- La tabla `bytes`
			- El usuario de la base de datos
			- La contraseña de la base de datos
			- La ruta local donde se almacenan los archivos en formato PARQUET

	- *Batch Layer*. Procesa en modo batch los datos almacenados en local en formato PARQUET. Se compone de:
		- Interfaz BatchJob.scala. El funcionamiento es el mismo que el de la Speed Layer.
		- Implementación de la interfaz AntennaBatchJob.scala, que extiende la interfaz BatchJob.scala. El funcionamiento es el mismo que el de la Speed Layer. Los argumentos de entrada que se le pasan son los siguientes:
			- Fecha/hora en formato ISO (p.ej. 2021-09-26T17:00:00+01:00)
			- La ruta local donde se encuentran los archivos almacenados en formato PARQUET.
			- La URI de conexión al servidor de PostgreSQL en Google Cloud
			- La tabla `user_metadata`
			- La tabla `bytes_hourly`
			- La tabla `user_quota_limit`
			- El usuario de la base de datos
			- La contraseña de la base de datos



