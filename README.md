# Integración Hadoop-ElasticSearch

## Descripción del Proyecto

Este proyecto demuestra la integración entre Hadoop y ElasticSearch, creando una conexión funcional entre Hive y un índice de ElasticSearch. La combinación de estas tecnologías permite aprovechar lo mejor de ambos mundos: el procesamiento masivo de datos de Hadoop y las capacidades de búsqueda y análisis en tiempo real de ElasticSearch.

## Beneficios de la Integración Hadoop-ElasticSearch

- **Análisis de Datos en Tiempo Real**: ElasticSearch permite realizar búsquedas y análisis de texto completo de manera eficiente, permitiendo obtener resultados rápidos en consultas complejas sobre grandes volúmenes de datos.

- **Integración de Fuentes de Datos**: Hadoop procesa y almacena grandes volúmenes de datos estructurados y no estructurados, que ahora pueden ser indexados en ElasticSearch para facilitar su acceso y búsqueda.

- **Indexación Eficiente**: Utilizamos Hadoop para preparar y procesar datos antes de indexarlos en ElasticSearch, aplicando transformaciones complejas o agregaciones masivas cuando sea necesario.

- **Consultas y Visualizaciones Avanzadas**: ElasticSearch proporciona capacidades avanzadas para consultas y visualizaciones, que ahora pueden ejecutarse sobre grandes conjuntos de datos procesados por Hadoop.

- **Búsqueda y Recuperación Optimizada**: Esta integración mejora significativamente la capacidad de búsqueda y recuperación de información, permitiendo encontrar rápidamente datos relevantes.

Al agregar la visualización con Kibana, obtenemos una solución completa para la obtención, procesamiento y gestión de Big Data.

## Arquitectura del Sistema

Nuestra arquitectura combina:
- Un clúster Hadoop en Google Cloud Dataproc
- Un servidor ElasticSearch (ELK stack) con Kibana y Logstash
- Conectores ES-Hadoop para la integración entre ambos sistemas

## Implementación Paso a Paso

### Parte 1: Configuración ES-Hadoop

#### Configuración del Clúster Hadoop
1. Creamos un clúster Hadoop en Google Cloud Dataproc
2. Cargamos las bibliotecas necesarias para la integración:
   - elasticsearch-hadoop-8.14.1.jar
   - commons-httpclient-3.1.jar
3. Creamos un bucket en Google Storage para almacenar estos archivos JAR
4. Descargamos los archivos desde el bucket al sistema de archivos del clúster

```bash
gsutil cp gs://bucket-para-elastic/jars/elastic/elasticsearch-hadoop-8.14.1.jar .
gsutil cp gs://bucket-para-elastic/jars/elastic/commons-httpclient-3.1.jar .
```

### Parte 2: Configuración del Servidor ElasticSearch

1. Creamos una máquina virtual para ElasticSearch en Google Cloud
2. Instalamos y configuramos ElasticSearch, desactivando las funciones de seguridad para esta prueba
3. Abrimos los puertos de firewall necesarios:
   - Puerto 9200 para ElasticSearch
   - Puerto 5601 para Kibana
4. Verificamos la conectividad entre el clúster Hadoop y el servidor ElasticSearch

### Parte 3: Configuración de la Conexión en el Clúster Hadoop

Modificamos la configuración de Hive para establecer la conexión con ElasticSearch:

```bash
sudo sed -i '$d' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <name>es.nodes</name>\n <value>IP_ELASTIC_SERVER</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <name>es.port</name>\n <value>9200</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <name>es.nodes.wan.only</name>\n <value>true</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo sed -i '$a \ <property>\n <name>hive.aux.jars.path</name>\n <value>/usr/lib/hive/lib/elasticsearch-hadoop-8.14.1.jar,/usr/lib/hive/lib/commons-httpclient-3.1.jar</value>\n </property>' /etc/hive/conf.dist/hive-site.xml
sudo cp elasticsearch-hadoop-8.14.1.jar /usr/lib/hive/lib/
sudo cp commons-httpclient-3.1.jar /usr/lib/hive/lib/
```

Estos comandos:
- Eliminan la última línea del archivo de configuración de Hive
- Agregan propiedades de configuración para la conexión con ElasticSearch
- Copian los JAR necesarios al directorio de bibliotecas de Hive

Finalmente, reiniciamos Hive para aplicar los cambios.

### Parte 4: Conexión de Datos

1. Creamos un índice en ElasticSearch desde el servidor:

```bash
curl -X POST "localhost:9200/alumnos/_doc/6" -H 'Content-Type: application/json' -d'
{
  "title": "New Document",
  "content": "This is a new document for the master class",
  "tag": ["general", "testing"]
}
'
```

2. Agregamos documentos al índice desde el clúster Hadoop:

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

3. Verificamos la inserción de datos con una consulta:

```bash
curl -X GET "http://IP-SERVER-ELASTIC:9200/alumnos/_search?pretty"
```

### Parte 5: Visualización con Kibana

1. Accedemos a Kibana a través del navegador: `http://IP_ELASTIC_SERVER:5601`
2. Creamos una vista de datos (Data View) para el índice "alumnos"
3. Creamos un dashboard con visualizaciones de los datos
4. Exploramos las diferentes opciones de visualización que ofrece Kibana

## Problemas Comunes y Soluciones

Durante la implementación de este proyecto, nos encontramos con varios desafíos técnicos que pudimos resolver:

### 1. Problemas de Conectividad entre Hadoop y ElasticSearch

**Problema:** El clúster Hadoop no podía conectarse al servidor ElasticSearch.

**Solución:** 
- Verificamos que las reglas de firewall estuvieran correctamente configuradas para permitir el tráfico al puerto 9200.
- Configuramos ElasticSearch en modo inseguro (`xpack.security.enabled: false`) para propósitos de prueba.
- Agregamos la propiedad `es.nodes.wan.only=true` en la configuración de Hive para permitir conexiones desde fuera de la red local.

### 2. Errores en la Carga de JAR

**Problema:** Hive no podía encontrar las bibliotecas de ElasticSearch-Hadoop.

**Solución:**
- Verificamos la ruta correcta donde debían colocarse los archivos JAR (`/usr/lib/hive/lib/`).
- Nos aseguramos de cargar el JAR correcto del paquete ElasticSearch-Hadoop (el que se encuentra en la carpeta `dist` del archivo ZIP).
- Reiniciamos el servicio Hive después de cargar los JAR.

### 3. Configuración Incorrecta de Hive

**Problema:** Los comandos `sed` para modificar el archivo de configuración `hive-site.xml` no funcionaban correctamente.

**Solución:**
- Editamos manualmente el archivo para asegurarnos de que la sintaxis XML fuera correcta.
- Verificamos que no hubiera caracteres especiales o espacios que pudieran afectar la interpretación de los comandos.
- Comprobamos que el archivo terminara correctamente con la etiqueta `</configuration>`.

### 4. Versiones Incompatibles

**Problema:** Incompatibilidad entre versiones de ElasticSearch y el conector ES-Hadoop.

**Solución:**
- Aseguramos que la versión de ElasticSearch (8.14.1) coincidiera con la versión del conector ES-Hadoop.
- Utilizamos la biblioteca commons-httpclient adecuada para evitar problemas de compatibilidad.

### 5. Problemas con la Indexación de Datos

**Problema:** Los datos no aparecían en ElasticSearch después de intentar indexarlos.

**Solución:**
- Verificamos la sintaxis correcta de los comandos curl para la inserción de datos.
- Comprobamos que el índice "alumnos" estuviera correctamente creado antes de intentar insertar documentos.
- Utilizamos el parámetro "pretty" en las consultas para facilitar la lectura de resultados y depuración.

### 6. Problemas con Kibana

**Problema:** Kibana no mostraba el índice "alumnos" en la lista de índices disponibles.

**Solución:**
- Refrescamos manualmente la lista de índices en Kibana.
- Creamos una vista de datos (Data View) escribiendo manualmente el patrón del índice.
- Verificamos que Kibana estuviera utilizando la configuración correcta para conectarse a ElasticSearch.

Estos desafíos son comunes en implementaciones de integración de tecnologías de Big Data y su resolución nos permitió ganar experiencia valiosa en la configuración y optimización de estos sistemas.

## Conclusiones

La integración de Hadoop con ElasticSearch proporciona una solución poderosa para el procesamiento, análisis y visualización de grandes volúmenes de datos. Esta configuración permite:

- Procesar datos masivos con Hadoop
- Indexar y buscar eficientemente con ElasticSearch
- Visualizar los resultados de manera interactiva con Kibana

Esta combinación es ideal para casos de uso como análisis de logs, búsqueda de texto completo en grandes repositorios de documentos, análisis de datos en tiempo real y muchas otras aplicaciones de Big Data.

## Requisitos Técnicos

- Google Cloud Platform (para Dataproc y VM)
- Hadoop/Hive
- ElasticSearch 8.x
- Kibana
- Conectores ES-Hadoop
- Acceso a firewall para configurar puertos

## Próximos Pasos

- Implementar seguridad en ElasticSearch
- Automatizar el proceso de ingesta de datos
- Crear pipelines de procesamiento más complejos
- Explorar capacidades avanzadas de agregación y visualización
