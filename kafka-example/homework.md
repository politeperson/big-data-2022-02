# Ejemplo de conectores con Apache Kafka  
En este tutorial utilizaremos los conectores por defecto que vienen en [Apache Kafka](https://kafka.apache.org/), el caso de uso que se implementará será el siguiente.  
## Caso de Uso  
El caso de uso a tratar consiste en detectar cambios que se realizan en un documento de texto `test.txt`, una vez Kafka detecta esos cambios lo que realizará es escribir esos cambios en otro documento de texto `test.sink.txt`.  
## Configuración  
Lo primero es instalar Apache Kafka, una vez descargado el archivo binario la estructura de nuestro directorio será la siguiente:  
.  
├── **bin**  
├── **config**    
├── **libs**  
├── LICENSE  
├── **licenses**  
├── **logs**  
├── NOTICE  
└── **site-docs**  
&emsp;&emsp;└── kafka_2.13-3.3.1-site-docs.tgz

lo primero será inicializar Zookeeper para el control del cluster, ubíquese en la raíz de la carpeta descargada y ejecute el siguiente comando:  
```bash
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```
luego inicializamos Kafka con el comando:  
```bash
$ bin/kafka-server-start.sh config/server.properties
```
Después ejecutaremos un _subscriber_ para ver en tiempo real los mensajes que se van publicando:  
```bash
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
```  
_Nota:_ El nombre del tópico al cual se suscribe es __connect-test__.  

## Configuración del _source_ de nuestros conectores  
Ahora pasaremos a configurar la fuente de nuestros conectores para habilitar el caso de uso.  
Primero creamos un documento llamado `source.properties` en el directorio de nuestra preferencia y le agregamos los siguientes datos. En mi caso yo lo haré en el directorio: `~/Desktop/kafka-examples`.  
```
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=/home/saul/Desktop/kafka-examples/test.txt
topic=connect-test
```
El documento de configuración es sencillo de entender, la propiedad _name_ es el nombre del conector, _connector.class_ es la clase java que viene por defecto y que ejecutaremos, el número máximo de tareas es 1, _file_ debe tener el _path_ de nuestro archivo de texto en mi caso ese es el path (`pwd` en la terminal para ver esa dirección), por último _topic_ que tiene el nombre del tópico que configuramos anteriormente.  
En el mismo directorio crearemos el archivo de texto `test.txt` y le llenamos con el texto que queramos.    

Luego crearemos la configuración para el segundo conector el cual escribirá en el archivo `test.sink.txt` los cambios que se realicen en `test.txt`, en mi caso lo creo en el mismo directorio, la configuración es similar al del primer conector, creamo el archivo `sink.properties` y agregamos.
```
name=local-file-sink
connector.class=FileStreamSink
tasks.max=1
file=/home/saul/Desktop/kafka-examples/test.sink.txt
topics=connect-test
```
Lo único que cambia es el nombre del conector y el archivo el cual escribirá.

## Iniciando la conexión
Para empezar la ejecución ejecutamos el siguiente comando.  
```
$ bin/connect-standalone.sh config/connect-standalone.properties ~/Desktop/kafka-examples/source.properties ~/Desktop/kafka-examples/sink.properties
```
Los dos últimos parámetros son la ubicación de los dos conectores que configuramos anteriormente.  
Una vez hecho esto podemos modificar el archivo `test.txt` y veremos en tiempo real como cambia el contenido de `test.sink.txt`, aquí algunas capturas.  

![kafka-consumer](https://i.imgur.com/aASvqRP.png)  
*En nuestro archivo `test.txt` hicimos todos esos cambios los cuales se escribiron en `test.sink.txt`*

![test.sink.txt](https://i.imgur.com/h1ZNmqt.png)  
*El resultado que se escribe en `test.sink.txt`*  
## Arquitectura del proyecto  
Lo que básicamente se hizo fué representar lo siguiente.  
![arquitectura](https://www.albertcoronado.com/wp-content/uploads/2020/08/kafka_connect.jpg)  
Se tienen dos fuentes de datos (los archivos txt) y lo que hacemos es usar los conectores para el caso de uso.