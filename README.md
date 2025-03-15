# Integración Hadoop-ElasticSearch

## Descripción del Proyecto

En este proyecto he demostrado la integración entre Hadoop y ElasticSearch, creando una conexión funcional entre Hive y un índice de ElasticSearch. La combinación de estas tecnologías me permitió aprovechar lo mejor de ambos mundos: el procesamiento masivo de datos de Hadoop y las capacidades de búsqueda y análisis en tiempo real de ElasticSearch.

## Requisitos Técnicos y Software Utilizado

Google Cloud Platform (GCP)
- Dataproc versión 2.1
- Hadoop 3.3.4
- Máquina virtual: e2-standard-4 (4 vCPUs, 16 GB memoria)
- Sistema operativo: Debian 11
  
Stack ELK
- ElasticSearch 8.14.1
- Logstash 8.14.1
- Kibana 8.14.1

Hadoop Ecosystem
- Hive 3.1.3

Librerías y Conectores
- ES-Hadoop 8.14.1
- Commons-httpclient 3.1

Herramientas para Documentación
- Claude Sonnet 3.7 - Utilizado para generar documentación detallada del proceso, problemas encontrados y soluciones

## Arquitectura del Sistema

Mi arquitectura combina:
- Un clúster Hadoop en Google Cloud Dataproc
- Un servidor ElasticSearch (ELK stack) con Kibana y Logstash
- Conectores ES-Hadoop para la integración entre ambos sistemas

## Implementación Paso a Paso

### Parte 1: Configuración ES-Hadoop

#### Configuración del Clúster Hadoop
1. Creé un clúster Hadoop en Google Cloud Dataproc
2. Cargué las bibliotecas necesarias para la integración:
   - elasticsearch-hadoop-8.14.1.jar
   - commons-httpclient-3.1.jar
3. Creé un bucket en Google Storage para almacenar estos archivos JAR
4. Descargué los archivos desde el bucket al sistema de archivos del clúster

```bash
gsutil cp gs://bucket-para-elastic/jars/elastic/elasticsearch-hadoop-8.14.1.jar .
gsutil cp gs://bucket-para-elastic/jars/elastic/commons-httpclient-3.1.jar .
```
![2025-03-06_23-52-14](https://github.com/user-attachments/assets/4bf728cc-736f-4761-9b14-2e9e305e7453)
*Fig. 1: Acá creé el bucket* 
![2025-03-06_23-56-13](https://github.com/user-attachments/assets/0800e70f-8607-4bb7-bd77-50c7e8716980)
*Fig. 2: Archivos cargados de acuerdo a las instrucciones de la práctica*
![2025-03-07_00-19-10](https://github.com/user-attachments/assets/11a47962-5db8-4163-84dc-dfafd3da95a4)
*Fig. 3: captura de pantalla de las instancias de VM en Google Cloud Console.*

### Parte 2: Configuración del Servidor ElasticSearch

1. Creé una máquina virtual para ElasticSearch en Google Cloud
2. Instalé y configuré ElasticSearch, desactivando las funciones de seguridad para esta prueba
3. Abrí los puertos de firewall necesarios:
   - Puerto 9200 para ElasticSearch
   - Puerto 5601 para Kibana
4. Verifiqué la conectividad entre el clúster Hadoop y el servidor ElasticSearch

![2025-03-09_00-21-33](https://github.com/user-attachments/assets/0d2a4b21-a1cc-4829-b32d-0cdf95e54822)
*Fig. 4: Configuración del archivo elasticsearch.yml con seguridad deshabilitada para permitir conexiones desde Hadoop*

Como se puede observar en la configuración, establecí los siguientes parámetros clave:
- `xpack.security.enabled: false` - Desactivé la seguridad para esta prueba de concepto
- `network.host: 0.0.0.0` - Configuré el servidor para aceptar conexiones desde cualquier dirección IP
- `http.port: 9200` - Mantuve el puerto estándar de ElasticSearch
- `discovery.type: single-node` - Configuré un único nodo para simplificar la implementación


### Parte 3: Configuración de la Conexión en el Clúster Hadoop

Modifiqué la configuración de Hive para establecer la conexión con ElasticSearch:

```bash
sudo sed -i '$d' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <n>es.nodes</n>\n <value>IP_ELASTIC_SERVER</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <n>es.port</n>\n <value>9200</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <n>es.nodes.wan.only</n>\n <value>true</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <n>hive.aux.jars.path</n>\n <value>/usr/lib/hive/lib/elasticsearch-hadoop-8.14.1.jar,/usr/lib/hive/lib/commons-httpclient-3.1.jar</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo cp elasticsearch-hadoop-8.14.1.jar /usr/lib/hive/lib/
sudo cp commons-httpclient-3.1.jar /usr/lib/hive/lib/
```

Estos comandos me permitieron:
- Eliminar la última línea del archivo de configuración de Hive
- Agregar propiedades de configuración para la conexión con ElasticSearch
- Copiar los JAR necesarios al directorio de bibliotecas de Hive

![2025-03-07_01-19-46](https://github.com/user-attachments/assets/1af4e32e-30e8-4039-9a8d-8dbd2339a235)
*Fig. 5: Proceso de instalación y configuración del servicio Elasticsearch en el clúster Hadoop*

Como se ve en la imagen, instalé y configuré Elasticsearch 8.17.3 en el clúster, habilitando y arrancando el servicio para que esté disponible para las conexiones desde Hive. La configuración del servicio se realizó mediante systemctl, asegurando que Elasticsearch se ejecute automáticamente al iniciar el sistema.

![2025-03-07_01-21-59](https://github.com/user-attachments/assets/b84bd60f-17a3-42a3-9383-ca3696ad5957)
*Fig. 6: Esta imagen muestra la interfaz de Google Cloud Console con la sección de Instancias de VM (máquinas virtuales) del proyecto "BigdataArquitectura"*

![2025-03-07_01-51-41](https://github.com/user-attachments/assets/eb037914-655d-46c0-b0f5-c738069f366d)
*Fig.7: Esta imagen muestra la consola SSH conectada a la instancia "belegost-m" (el nodo maestro del clúster Hadoop) en Google Cloud.* 

Se puede observar el estado del servicio Hive Metastore, que es un componente crítico para la integración entre Hadoop y ElasticSearch.
Específicamente, la imagen muestra:

La salida del comando systemctl status hive-metastore.service, que indica que el servicio Hive Metastore está:
- Cargado (Loaded: loaded)
- Activo y en ejecución (Active: active (running)) desde el 7 de marzo de 2025
- Ejecutándose con el proceso principal PID 207476 (Java)

Los logs de inicio del servicio que muestran:
- El arranque exitoso del servicio Hive Metastore
- El inicio de sesión como usuario "hive"
- La confirmación final "Started hive-metastore.service - LSB: Hive Metastore"

Esta imagen documenta que el servicio Hive Metastore está correctamente configurado y funcionando en el nodo maestro del clúster Hadoop, lo cual es esencial para que Hive pueda comunicarse con ElasticSearch a través de los conectores ES-Hadoop.


### Parte 4: Conexión de Datos

1. Creé un índice en ElasticSearch desde el servidor:

```bash
curl -X POST "localhost:9200/alumnos/_doc/6" -H 'Content-Type: application/json' -d'
{
  "title": "New Document",
  "content": "This is a new document for the master class",
  "tag": ["general", "testing"]
}
'
```

2. Agregué documentos al índice desde el clúster Hadoop:

```bash
curl -X POST "IP-SERVER-ELASTICSEARCH:9200/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_index": "alumnos", "_id": "3" } }
{ "id": 3, "name": "Carlos", "last_name": "González" }
{ "index": { "_index": "alumnos", "_id": "4" } }
{ "id": 4, "name": "María", "last_name": "López" }
{ "index": { "_index": "alumnos", "_id": "5" } }
{ "id": 5, "name": "Luis", "last_name": "Martínez" }
{ "index": { "_index": "alumnos", "_id": "7" } }
{ "id": 7, "name": "Sofía", "last_name": "Ramírez" }
{ "index": { "_index": "alumnos", "_id": "8" } }
{ "id": 8, "name": "Pedro", "last_name": "Hernández" }
'
```

3. Verifiqué la inserción de datos con una consulta:

```bash
curl -X GET "http://IP-SERVER-ELASTIC:9200/alumnos/_search?pretty"
```
![2025-03-07_01-51-41](https://github.com/user-attachments/assets/cb6718a2-c6f4-46cb-bb0c-010710dbb0e9)
*Fig 8: conexión exitosa entre el clúster Hadoop y ElasticSearch*

La ejecución del comando curl -X GET "http://10.128.0.6:9200/alumnos/_search?pretty", que realiza una consulta al índice "alumnos" en el servidor ElasticSearch (con IP 10.128.0.6 en el puerto 9200). El resultado de la consulta en formato JSON, que muestra:

- Un código de estado HTTP 200 (éxito)
- 6 resultados ("hits" con "value": 6)
- Los datos de varios documentos del índice "alumnos", incluyendo:
  - Un documento con ID "6" que contiene un "title": "New Document" y "content": "This is a new document for the master class"
  - Un documento con ID "3" para "Carlos González"
  - Un documento con ID "4" para "María López"
  - Un documento con ID "5" (se ve parcialmente)


### Parte 5: Visualización con Kibana

1. Accedí a Kibana a través del navegador: `http://IP_ELASTIC_SERVER:5601` (previa instalación de la aplicación)
2. Creé una vista de datos (Data View) para el índice "alumnos"
3. Creé un dashboard con visualizaciones de los datos
4. Exploré las diferentes opciones de visualización que ofrece Kibana

![2025-03-08_23-36-23](https://github.com/user-attachments/assets/0135762a-a210-42fd-8fef-2ce3a3629aa3)
*Fig 9: Instalación de Kibana*

En la imagen se puede observar:

La ejecución del comando sudo systemctl status kibana que muestra que el servicio de Kibana está:

- Cargado correctamente (Loaded: loaded)
- Activo y en ejecución (Active: active (running)) desde el 9 de marzo de 2025
- Con un PID principal de 6404 ejecutándose como un proceso Node.js

Los logs de Kibana que muestran un funcionamiento normal, con mensajes INFO sobre diferentes plugins e inicialización del servicio.
La verificación de que el directorio de logs existe y tiene los permisos correctos (con el comando sudo mkdir -p /var/log/kibana).
La confirmación de que Kibana está escuchando en el puerto 5601 (verificado con netstat -tulpn | grep 5601), que muestra:
Copytcp 0 0 0.0.0.0:5601 0.0.0.0:* LISTEN

Esto indica que Kibana está correctamente configurado, ejecutándose y escuchando en el puerto 5601 en todas las interfaces de red (0.0.0.0), lo que significa que está accesible desde fuera del servidor.

<img width="1298" alt="captura (1)" src="https://github.com/user-attachments/assets/871a0ff8-88b7-4357-8cd3-d651a3bebbca" />
*Fig 10: Dashboard en blanco, pero con los datos listos para crear visualizaciones*

![2025-03-08_23-56-38](https://github.com/user-attachments/assets/85b46260-e78c-4071-995f-7097f8a87595)
*Fig 11: La imagen muestra un dashboard de Kibana que has creado con múltiples visualizaciones de los datos almacenados en el índice "alumnos" de ElasticSearch*

El dashboard contiente: 
- Un gráfico circular que muestra la "distribución de los tags" (50% general, 50% testing)
- Una tabla titulada "Contador" que muestra:
 - Top 5 valores de id
 - Top 3 valores de apellidos (last_name.keyword)
 - Top 3 valores de nombres (name.keyword)
 - Conteo de registros
- Una sección de "nombres" con una nube de palabras que muestra los nombres de los alumnos (Carlos, Luis, Sofía, Pedro, María)
- Un gráfico de barras titulado "conteo único" que muestra el conteo único de IDs



## Problemas que Enfrenté y Cómo los Resolví

Durante la implementación de este proyecto, me encontré con varios desafíos técnicos:

### 1. Problemas de Conectividad entre Hadoop y ElasticSearch

**Problema:** Mi clúster Hadoop no podía conectarse al servidor ElasticSearch.

**Solución:** 
- Verifiqué que las reglas de firewall estuvieran correctamente configuradas para permitir el tráfico al puerto 9200.
- Configuré ElasticSearch en modo inseguro (`xpack.security.enabled: false`) para propósitos de prueba.
- Agregué la propiedad `es.nodes.wan.only=true` en la configuración de Hive para permitir conexiones desde fuera de la red local.

### 2. Errores en la Carga de JAR

**Problema:** Hive no podía encontrar las bibliotecas de ElasticSearch-Hadoop.

**Solución:**
- Verifiqué la ruta correcta donde debían colocarse los archivos JAR (`/usr/lib/hive/lib/`).
- Me aseguré de cargar el JAR correcto del paquete ElasticSearch-Hadoop (el que se encuentra en la carpeta `dist` del archivo ZIP).
- Reinicié el servicio Hive después de cargar los JAR.

### 3. Configuración Incorrecta de Hive

**Problema:** Los comandos `sed` para modificar el archivo de configuración `hive-site.xml` no funcionaban correctamente.

**Solución:**
- Edité manualmente el archivo para asegurarme de que la sintaxis XML fuera correcta.
- Verifiqué que no hubiera caracteres especiales o espacios que pudieran afectar la interpretación de los comandos.
- Comprobé que el archivo terminara correctamente con la etiqueta `</configuration>`.

### 4. Versiones Incompatibles

**Problema:** Tuve incompatibilidad entre versiones de ElasticSearch y el conector ES-Hadoop.

**Solución:**
- Me aseguré de que la versión de ElasticSearch (8.14.1) coincidiera con la versión del conector ES-Hadoop.
- Utilicé la biblioteca commons-httpclient adecuada para evitar problemas de compatibilidad.

### 5. Problemas con la Indexación de Datos

**Problema:** Los datos no aparecían en ElasticSearch después de intentar indexarlos.

**Solución:**
- Verifiqué la sintaxis correcta de los comandos curl para la inserción de datos.
- Comprobé que el índice "alumnos" estuviera correctamente creado antes de intentar insertar documentos.
- Utilicé el parámetro "pretty" en las consultas para facilitar la lectura de resultados y depuración.

### 6. Problemas con Kibana

**Problema:** Kibana no mostraba el índice "alumnos" en la lista de índices disponibles.

**Solución:**
- Refresqué manualmente la lista de índices en Kibana.
- Creé una vista de datos (Data View) escribiendo manualmente el patrón del índice.
- Verifiqué que Kibana estuviera utilizando la configuración correcta para conectarse a ElasticSearch.

Enfrentar y superar estos desafíos me permitió ganar experiencia valiosa en la configuración y optimización de estos sistemas de Big Data.

## Conclusiones

La integración de Hadoop con ElasticSearch me proporcionó una solución poderosa para el procesamiento, análisis y visualización de grandes volúmenes de datos. Con esta configuración pude:

- Procesar datos masivos con Hadoop
- Indexar y buscar eficientemente con ElasticSearch
- Visualizar los resultados de manera interactiva con Kibana

Esta combinación resultó ideal para casos de uso como análisis de logs, búsqueda de texto completo en grandes repositorios de documentos, análisis de datos en tiempo real y muchas otras aplicaciones de Big Data.


Elaborado por Ulises González en la ciudad de Panamá, los 15 días del mes de marzo de 2025