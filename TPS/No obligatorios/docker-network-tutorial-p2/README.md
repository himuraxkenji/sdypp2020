# Tutorial parte 2 
## Sección 1 -- Escalar Docker con Docker compose

Bienvenidos a la segunda parte del tutorial de Docker network. Este tutorial te guiará en:
- Continuar avanzando con los conceptos para trabajar con tus propias imágenes
- Continuar configurando contenedores utilizando Dockerfiles
- Trabajar con volúmenes compartidos (en Host) para persistir elementos en contenedores y actualizar cambios
- Comprender como pasar parámetros a los contenedores (ajustando el código java) desde DockerCompose
- Diseñar e implementar servicios distribuidos sobre Docker 
- Implementar un balanceador de cargas en el Host para redireccionar peticiones

## Sección 2 -- Manejo de parámetros en DockerCompose
Existen varias maneras para pasar parámetros a las aplicaciones desde DockerCompose.
En este caso nosotros utilizaremos las variables de entorno (environment) 

```yaml
version: '3'
services:
  server:
    build: ./servidor
    ports:
      - "4444:4444"
    volumes:
      - /tmp/javadir:/tmp/javadir
    environment:
      - nombre=Servidor 1
      - logName=logfile1
      - port=4444
```

## Sección 3 -- Crear varias instancias para un mismo servicio

## Sección 4 -- Balancear carga entre instancias de servicios con HaProxy

## Sección 5 -- Construir un balanceador de carga redundante con HaProxy

## Sección 6 -- Crear un cluster RabbitMQ con Docker Compose

### 6.1 - Qué es RabbitMQ?

> RabbitMQ es un "open source message broker software" que implementa los protocolos de "Advanced Message Queuing Protocol (AMQP)".
> El servicio de RabbitMQ está escrito en Erlang y construido por Open Telecom Platform
> Tiene la capacidad de construir su servicio como un cluster con capacidad de redundancia ante fallas. Están disponibles  
> API's para varios lenguajes de programación

https://www.rabbitmq.com/

### 6.2 - RabbitMQ en Docker
La imagen oficial de RabbitMQ se encuentra desarrollada y presentada (desde Marzo de 2020) por Bitnami Inc y se encuentra preparada para trabajar en diversos entornos (local, Cloud, Kubernetes, etc)
https://bitnami.com/stack/rabbitmq/containers

La imagen base se encuentra en Docker Hub, donde se puede visualizar el contínuo update de la misma. 
https://hub.docker.com/r/bitnami/rabbitmq/

### 6.3 - Porqué usar las imágenes Bitnami

* Bitnami monitorea contínuamente los cambios que producen los proveedores de las plataformas y publica rápidamente las nuveras versiones de esas imágenes a través de pipelines automatizados 
* Con las imágenes de Bitnami, las últimas correcciones de errores y características están disponibles lo antes posible.
* Los contenedores de Bitnami, las máquinas virtuales y las imágenes en la nube utilizan los mismos componentes y el mismo enfoque de configuración, lo que facilita el cambio entre formatos según las necesidades de su proyecto.
* Todas las imágenes de bitnami están basadas en [minideb] (https://github.com/bitnami/minideb) que es una imagen de contenedor minimalista de tamaño reducido basada en Debian (distribución líder de Linux) .
* Todas las imágenes de Bitnami disponibles en Docker Hub están firmadas con [Docker Content Trust (DTC)] (https://docs.docker.com/engine/security/trust/content_trust/). Puede usar `DOCKER_CONTENT_TRUST = 1` para verificar la integridad de las imágenes

Cómo siempre existen varias maneras de crear un cluster de un determinado servicio para ofrecer características de redundancia, tolerancia a fallos y replicación. Especialmente en docker existen siempre diversos usuarios que publican adaptaciones de las imágenes oficiales para funciones específicas o facilitar el uso de algún plugin o herramienta.  Sin embargo, este tipo de adaptaciones casi siempre quedan sin actualizaciones frecuentes y terminan ofreciendo herramientas desactualizadas.  Por lo tanto, en este tutorial siempre se busca utilizar herramientas de repositorios oficiales con updates contínuos.

### 6.4 - Cómo correr RabbitMQ desde la imagen básica (docker) de Bitnami

* Obtener la imagen desde el repositorio
```bash
$ docker run --name rabbitmq bitnami/rabbitmq:latest

Unable to find image 'bitnami/rabbitmq:latest' locally
latest: Pulling from bitnami/rabbitmq
f741ee3cf64f: Pull complete 
6e1a7737956d: Pull complete 
ced5f76dedf2: Downloading [=============================================>     ]  14.34MB/15.83MB
....

Starting RabbitMQ 3.8.3 on Erlang 22.3
 Copyright (c) 2007-2020 Pivotal Software, Inc.
 Licensed under the MPL 1.1. Website: https://rabbitmq.com

  ##  ##      RabbitMQ 3.8.3
  ##  ##
  ##########  Copyright (c) 2007-2020 Pivotal Software, Inc.
.....
2020-04-13 15:22:53.106 [info] <0.337.0> Running boot step networking defined by app rabbit
2020-04-13 15:22:53.108 [info] <0.488.0> started TCP listener on [::]:5672
2020-04-13 15:22:53.108 [info] <0.337.0> Running boot step cluster_name defined by app rabbit
2020-04-13 15:22:53.108 [info] <0.337.0> Running boot step direct_client defined by app rabbit
2020-04-13 15:22:53.109 [info] <0.337.0> Running boot step os_signal_handler defined by app rabbit
2020-04-13 15:22:53.109 [info] <0.490.0> Swapping OS signal event handler (erl_signal_server) for our own
2020-04-13 15:22:53.149 [info] <0.540.0> Management plugin: HTTP (non-TLS) listener started on port 15672
2020-04-13 15:22:53.149 [info] <0.646.0> Statistics database started.
.....
```

* Verificar (en otra terminal) que está corriendo el contenedor
```bash
$ docker container ps
```
```
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                      NAMES
7daa75d3e3db        bitnami/rabbitmq:latest   "/opt/bitnami/script…"   12 seconds ago      Up 10 seconds       4369/tcp, 5672/tcp, 15672/tcp, 25672/tcp   rabbitmq
```
* Parar el servicio (Ctrl + c)

#### 6.5 - Usar Docker Compose para correr RabbitMQ 

* Descargar el "template" del docker-compose disponible por el proveedor y visualizarlo 

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-rabbitmq/master/docker-compose.yml > docker-compose.yml
cat docker-compose.yml
```
```yaml
version: '3'

services:
  rabbitmq:
    image: 'bitnami/rabbitmq:latest'
    ports:
      - '4369:4369'
      - '5672:5672'
      - '25672:25672'
      - '15672:15672'
    volumes:
      - 'rabbitmq_data:/bitnami'
volumes:
  rabbitmq_data:
    driver: local
```

* Hacer una prueba y levantarlo con docker-compose
```bash
$ docker-compose up -d
```

#### 6.6 - Persistir los datos de RabbitMQ (a pesar de reinicio) 

Si se elimina el contenedor, todos sus datos se perderán, y la próxima vez que ejecute la imagen, la base de datos se reiniciará. Para evitar esta pérdida de datos, debe montar un volumen que persistirá incluso después de quitar el contenedor.

Para persistencia, debe montar un directorio en la ruta `/bitnami`. Si el directorio montado está vacío, se inicializará en la primera ejecución.

```bash
$ docker run \
    -v /path/to/rabbitmq-persistence:/bitnami \
    bitnami/rabbitmq:latest
```

También puede hacer esto con un cambio menor en el archivo [`docker-compose.yml`] (https://github.com/bitnami/bitnami-docker-rabbitmq/blob/master/docker-compose.yml) presente en este repositorio:

```yaml
rabbitmq:
  ...
  volumes:
    - /path/to/rabbitmq-persistence:/bitnami
  ...
```

#### 6.7 - Configuración avanzada de RabbitMQ (Docker)

##### Variables de entorno (Como vimos al principio)

Cuando se inicia la imagen de rabbitmq, se pueden ajustar configuraciones de la instancia pasando una o más variables de entorno en el archivo docker-compose o en la línea de comandos de ejecución de docker. Si desea agregar una nueva variable de entorno:

Para docker-compose, agregue el nombre y el valor de la variable en la sección de la aplicación en [`docker-compose.yml`] 

```yaml
rabbitmq:
  ...
  environment:
    - RABBITMQ_PASSWORD=my_password
  ...
```
* Variables disponibles:

 - `RABBITMQ_USERNAME`: RabbitMQ application username. Default: **user**
 - `RABBITMQ_PASSWORD`: RabbitMQ application password. Default: **bitnami**
 - `RABBITMQ_HASHED_PASSWORD`: RabbitMQ application hashed password.
 - `RABBITMQ_VHOST`: RabbitMQ application vhost. Default: **/**
 - `RABBITMQ_ERL_COOKIE`: Erlang cookie to determine whether different nodes are allowed to communicate with each other.
 - `RABBITMQ_NODE_TYPE`: Node Type. Valid values: *stats*, *queue-ram* or *queue-disc*. Default: **stats**
 - `RABBITMQ_NODE_NAME`: Node name and host. E.g.: *node@hostname* or *node* (localhost won't work in cluster topology). Default **rabbit@localhost**. If using this variable, ensure that you specify a valid host name as the container wil fail to start otherwise.
 - `RABBITMQ_NODE_PORT_NUMBER`: Node port. Default: **5672**
 - `RABBITMQ_CLUSTER_NODE_NAME`: Node name to cluster with. E.g.: **clusternode@hostname**
 - `RABBITMQ_CLUSTER_PARTITION_HANDLING`: Cluster partition recovery mechanism. Default: **ignore**
 - `RABBITMQ_MANAGER_PORT_NUMBER`: Manager port. Default: **15672**
 - `RABBITMQ_DISK_FREE_LIMIT`: Disk free space limit of the partition on which RabbitMQ is storing data. Default: **{mem_relative, 1.0}**
 - `RABBITMQ_ULIMIT_NOFILES`: Resources limits: maximum number of open file descriptors. Default: **65536**
 - `RABBITMQ_ENABLE_LDAP`: Enable the LDAP configuration. Defaults to `no`.
 - `RABBITMQ_LDAP_TLS`: Enable secure LDAP configuration. Defaults to `no`.
 - `RABBITMQ_LDAP_SERVER`: Hostname of the LDAP server. No defaults.
 - `RABBITMQ_LDAP_SERVER_PORT`: Port of the LDAP server. Defaults to `389`.
 - `RABBITMQ_LDAP_USER_DN_PATTERN`: DN used to bind to LDAP in the form `cn=$${username},dc=example,dc=org`. No defaults.

#### 6.8 - Construir un cluster RabbitMQ usando Docker Compose

This is the simplest way to run RabbitMQ with clustering configuration:

##### Paso 1: Crear en /tmp/ los "volúmenes" que usaremos para montar en los nodos 
Vamos a crear:
* 1 folder para el stats+node
* 1 folder para cada queue-disc
```bash
 mkdir /tmp/rabbit; mkdir /tmp/rabbit/stats ; mkdir /tmp/rabbit/node1 ; mkdir /tmp/rabbit/node2 ; mkdir /tmp/rabbit/node3; sudo chmod 777 -R /tmp/rabbit
```
##### Paso 2: Crear el primer nodo stats al docker-compose.yml`

* Copie el código a continuación en su docker-compose.yml para agregar un nodo de estadísticas web RabbitMQ a la configuración de su clúster.

```yaml
version: '3'

services:
  stats:
    image: bitnami/rabbitmq
    environment:
      - RABBITMQ_NODE_TYPE=stats
      - RABBITMQ_NODE_NAME=rabbit@stats
      - RABBITMQ_ERL_COOKIE=s3cr3tc00ki3
    ports:
      - '15672:15672'
    volumes:
      - '/tmp/rabbit/stats:/bitnami'
```

> ** Nota: ** El nombre del servicio (** stats **) es importante para que un nodo pueda resolver el nombre de host para luego poder agrupar los nodos. (Tenga en cuenta que el nombre del nodo es `rabbit@stats`)

* Verificar (además de lo informado por consola) que está corriendo el contenedor
```bash
$ docker container ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                  NAMES
223e1587c160        bitnami/rabbitmq    "/opt/bitnami/script…"   39 minutes ago      Up 25 minutes       4369/tcp, 5672/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp                docker-network-tutorial-p2_stats_1
```

* Validar ingresando a la web-gui que está corriendo el servicio
http://localhost:15672

##### Paso 3: Agregar nodos "colas" a la configuración del cluster RabbitMQ

Actualizar con uno o más nodos según el ejemplo siguiente 

```yaml
version: '3'
services:
  stats:
    image: bitnami/rabbitmq
    environment:
      - RABBITMQ_NODE_TYPE=stats
      - RABBITMQ_NODE_NAME=rabbit@stats
      - RABBITMQ_ERL_COOKIE=s3cr3tc00ki3
    ports:
      - '15672:15672'
    volumes:
      - '/tmp/rabbit/stats/:/bitnami'
  node1:
    image: bitnami/rabbitmq
    environment:
      - RABBITMQ_NODE_TYPE=queue-disc
      - RABBITMQ_NODE_NAME=rabbit@node1
      - RABBITMQ_CLUSTER_NODE_NAME=rabbit@stats
      - RABBITMQ_ERL_COOKIE=s3cr3tc00ki3
    ports:
      - 5672:5672
      - 4369:4369
    volumes:
      - '/tmp/rabbit/node1:/bitnami'
  node2:
    image: bitnami/rabbitmq
    environment:
      - RABBITMQ_NODE_TYPE=queue-disc
      - RABBITMQ_NODE_NAME=rabbit@node2
      - RABBITMQ_CLUSTER_NODE_NAME=rabbit@stats
      - RABBITMQ_ERL_COOKIE=s3cr3tc00ki3
    ports:
      - 5673:5672
      - 4370:4369
    volumes:
      - '/tmp/rabbit/node2:/bitnami'   
```

##### Paso 4: Parar el cluster y verificar las configuraciones 

Detener el cluster utilizando los siguientes comandos de Docker Compose: 

```bash
$ docker-compose stop rabbitmq
```
##### Paso 5: Realizar un backup de las configuraciones para disponer en otro nodo más adelante 
El backup se realizará a través de utilizar la herramienta rsync hacia otra carpeta utilizando

```bash
$ rsync -a /path/to/rabbitmq-persistence /path/to/rabbitmq-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```
