# Jenkins Workshop

Guía de la charla acerca de Jenkins y entregabilidad https://youtu.be/rpyTevjM3X4

## Parte 1: Docker

Utilizaremos Docker para gestionar builds, test y en último término plataformado y operación de entornos.

### Gestión de elementos

Trabajaremos con 4 diferentes elementos de docker:

* Imágenes
* Containers
* Redes
* Volúmenes

Existen opciones comunes a todos los elementos:

```bash
$ docker ${item} ls          # Listar todos los elementos de su tipo. Opcional "-a" en containers e imágenes
$ docker ${item} rm          # Borrar un elemento. Opcional "-f" para forzar el borrado
$ docker ${item} prune       # Borrar todos los elementos de un tipo. Opcional "-f" no pide confirmación.
```

Como ${item} podemos usar:

* image
* container
* network
* volume

Tenemos sistaxis específicas para elementos concretos, por ejemplo son equivalentes:

```bash
$ docker ps -a              # Equivalente a "docker container ls -a"
$ docker images             # Equivalente a "docker image ls"
$ docker rmi ubuntu         # Equivalente a "docker image rm ubuntu"
```

Hay opciones que podemos usar en algunos elementos, en otros no:

```bash
$ docker container ps       # Equivalente a "docker container ls -a"
$ docker image ps           # Incorrecto, con esto tendremos un error
```

Podemos obtener ayuda de docker para cada uno de los comandos con "docker help"

```bash
$ docker help container     # Listado de opciones de "docker container"
$ docker help container ps  # Listado de opciones de "docker container ps"
```

#### Imágenes

Podemos considerarlo como plantillas que tomaremos de base para ejecutar containers. en los ejemplos trabajaremos con el registry "docker hub".

Un registry es un repositorio público donde guardaremos nuestras imágenes.

```bash
$ docker pull redpandaci/jenkins-dind  # Podemos usar "docker image pull redpandaci/jenkins-dind"
$ docker pull hello-world:latest       # Descargamos la imagen dicker del clásico "Hello, world!" en su última versión
```

Cada vez que hacemos "docker pull" de una imagen:

* En caso que imagen no exista en nuestro PC, se descargará del registry
* Si la imagen ya la tenemos en nuesto PC, comprobará si la está actualizada, descargando la última versión en caso necesario

Podemos indicar una versión concreta a la hora de descargar una imagen:

```shell
$ docker pull redpandaci/jenkins-dind:2.89.4
```

En caso de no especificar versión, se bajará la última (latest)

```shell
$ docker pull redpandaci/jenkins-dind:latest    # Equivalente a "docker pull redpandaci/jenkins-dind"
```

#### Containers

Se pueden considerar "objetos" de la "clase" imagen.

```bash
$ docker run --rm hello-world                   # Ejecución del clásico "Hello, World!

[...]

$ docker container ps -a                        # Listado de containers, tanto que están en ejecución como los que finalizaron
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
64897191cb95        hello-world         "/hello"            1 second ago        Exited (0) 2 seconds ago                       unruffled_ardinghelli
$ docker container rm unruffled_ardinghelli     # Borrado de container
unruffled_ardinghelli
$ docker ps -a                                  # Atajo para "docker container ps -a"
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

#### Redes

Siguiendo la filosofia como gestión de elementos, podemos usar "docker network [comando]"

```shell
$ docker network ls                                                 # Listamos todas las redes
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
$ docker network create test                                        # Creamos una nueva red llamada "test"
6d2646754a9ebfb024c8f0e50df5dc31f4a8d78342c0173599c2fd33c9c9408a
$ docker network ls                                                 # Verificamos que la red se ha creado
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
6d2646754a9e        test                bridge              local
$ docker network inspect test                                       # Vemos las propiedades de la red "test"
[
    {
        "Name": "test",
        "Id": "6d2646754a9ebfb024c8f0e50df5dc31f4a8d78342c0173599c2fd33c9c9408a",
        "Created": "2018-02-22T16:23:52.08840954Z",

[...]

        "Labels": {}
    }
]
$ docker network rm test                                            # Borramos la red "test"
test
```

Con la visión "catle" de gestión de elementos (los elementos son "ganado"), siguiendo el ejemplo del vídeo vamos a crear y destruir con "prune" 10 redes

```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
$ for a in 1 2 3 4 5 6 7 8 9 10; do docker network create test_$a; done
50d89397a430f77ff6f506842dced6406f4e4097d1fbd0428ab2277059556d6c
a56cf873f879cc656b7f10bdb20450eff0866c019999a13229b15822d81a9092
e1c2a069eb34b5c509019e96a065099c0e6516fa2a2faeb71c4b6be5aa8c2545
75b6d460e36ea222d6373c35bdf762cc3fc3f1b3da21a29b1d2f2dbad57974bc
e0069de70155a9edad494e79025cd7d93880262fbafbf369578e581b3c9e4eec
6441aa7795c09f68d8e14c1923232ee0212dc1e3f78c3c9f4534cb14c45627b6
89db9380b8fe312fb16dc34c563c0c3ebdabfd423d9fe5ad94ea6c0b21d2c29b
6836e20be411799585661cdea6bbcfa2b29f021eaa6132ad44f39bdf049b0908
cdd4d068b596a82ffa725945e86aa3dcff29cf9fabfe175e83d2efdf16df17e9
a693514bf8d6599f4ee879f779e655b8bb95cf78acb5d298ed5b212e3da3e60a
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
50d89397a430        test_1              bridge              local
a693514bf8d6        test_10             bridge              local
a56cf873f879        test_2              bridge              local
e1c2a069eb34        test_3              bridge              local
75b6d460e36e        test_4              bridge              local
e0069de70155        test_5              bridge              local
6441aa7795c0        test_6              bridge              local
89db9380b8fe        test_7              bridge              local
6836e20be411        test_8              bridge              local
cdd4d068b596        test_9              bridge              local
$ docker network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Networks:
test_3
test_5
test_8
test_9
test_10
test_2
test_6
test_7
test_1
test_4

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
```

Podemos crear una red y ejecutar un container que la utilice
```shell
$ docker network create test
d3fc0ff8c1a473b83b78eadc9313b1aacaf2790b37e8cf40eb7f8c80bc1a1406
$ docker run --network test hello-world

Hello from Docker!

[...]

$ docker container inspect romantic_einstein
[
    {
        "Id": "b6343150a7301c9109fd501c1ba91191751c202596c7cd5eb4e386bc9f02d10e",
        "Created": "2018-02-22T16:36:46.07837747Z",

 [...]

            "Networks": {
                "test": {

[...]
```

#### Volúmenes

### Docker compose

## Parte 2: Jenkins

### Uso de Docker para montaje de Jenkins en local y dockerizado

### Configuración y uso de agentes (nodos)

### Gestión de plugins y configuración

### Creación de organización Github y "Engagement" a Jenkins

### Creación de projecto Bitbucket y "Engagement" a Jenkins

## Parte 3: Pipelines (enlazado con los dos últimos puntos de la parte 2)

### Uso de Jenkins Pipeline

### Configuración de job Android

### Configuración de job Back / Front

### Configuración de job iOS (en Jenkins QA)
