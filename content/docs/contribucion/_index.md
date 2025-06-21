---
title: Contribución
type: docs
toc: true
---


Esta sección es para establecer algunos estandares importantes que debe tener un repositorio para que este pueda funcionar en DEB y que sea includio.  
A continuación se mostrara algunos ejemplos utiles y esenciales para que puedan reutilizarlo y asi crear su paquete deb

## Requisitos importantes

- Todo paquete debe funcionar sobre la base de PluriOS (Ubuntu 24.04 LTS)
- El resultado (paquete deb) del proyecto no debe superar los 99MB
- Todo paquete debe ser compatible con las versiones de herramientas disponibles en el sistema o en su repositorio actual (Ubuntu 24.04 LTS) de PluriOS, ej: Python3, Perl, C/C++ y otros

## Plantillas

En esta sección se tendra una muestra de los archivos "plantillas" que se puede reutilizar para crear un **paquete DEB**

### DEBIAN (directorio)

En este directorio DEBIAN, es donde se encuentran los siguientes archivos comunes que todo paquete debe tener:
- **control**     : Información del paquete (nombre, versión, descripción, dependencias, mantenedor, etc.)
- **postinst**    : Script que se ejecuta **después** de instalar el paquete
- **preinst**     : Script que se ejecuta **antes** de instalar el paquete
- **postrm**      : Script que se ejecuta **después** de eliminar el paquete
- **prerm**       : Script que se ejecuta **antes** de eliminar el paquete
- **conffiles**   : Lista de archivos de configuración para que no se sobreescriban en una actualización
- **md5sums**     : Sumas MD5 de los archivos incluidos en el paquete



### control


```bash {filename=control}
Package: CHANGE_NAME
Version: CHANGE_VERSION
Architecture: CHANGE_ARCH
Section: misc
Priority: optional
Installed-Size: CHANGE_SIZE
Maintainer: AGREGAR NOMBRE <AGREGAR CORREO>
Copyright: AGREGAR LICENCIA
Description: INGRESAR DESCRIPCION
```

> Nota.- no cambiar las variables que dicen CHANGE_* , estas variables son utilizadas en build para reemplazar la información.  



#### Ejemplo 1


**Estructura de directorio**

La siguiente estructura es un ejemplo de como debe obtenerse en el directorio de trabajo para que pueda empaquetarse

```bash {filename=package}
mi-programa_0.3_all/
├── DEBIAN/
│   ├── control   # importante
│   ├── postinst  # opcional
│   ├── prerm     # opcional
│   └── md5sums   # opcional
└── usr/
    └── bin/
        └── mi-programa
```

**build**

El siguiente ejemplo es un script `build` que permite empaquetar un programa que no requiere descargar ningun recurso desde internet. Es decir, el repositorio debe tener todo lo necesario para que el paquete pueda instalarse y ejecutarse correctamente.

```bash {filename=build}
#!/bin/bash

# verifica si el script se ejecuta como root
if [ "$(id -u)" -ne 0 ]; then
  echo "This script must be run as root."
  exit 1
fi

# definir variables para el paquete
NAME="mi-programa"      # nombre del paquete : minusculas, usar el simbolo - en vez de espacio, sin acentos
VERSION="0.3"           # version: x.x o x.x.x (recomendable)
ARCH="amd64"            # arch: amd64, i386, all

# eliminar archivos no relevantes para subir a github
# estas instrucciones puede variar dependiendo los archivos innecesarios del proyecto
rm -rf *.deb *.tar.gz ${NAME}*${ARCH}* "${NAME}" 2>/dev/null

# Crear el nombre del directorio de trabajo 
# Ej: mi-programa_0.3_amd64
DIR_PACKAGE="${NAME}_${VERSION}_${ARCH}"

# Crea el directorio de trabajo 
mkdir -p $DIR_PACKAGE

# Copia y pega los directorios necesarios para el paquete
# DEBIAN es un directorio importante que contiene todos los metadatos de un paquete DEB
cp -r DEBIAN $DIR_PACKAGE/
cp -r usr/ $DIR_PACKAGE/

# Calcula el tamaño en kilobytes (kB) de todo el directorio de trabajo excluyendo el directorio DEBIAN
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk '{print $1}')

# Cambia información del archivo control como ser: nombre, versión, tamaño
sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SIZE/$SIZE/g" "$DIR_PACKAGE/DEBIAN/control"

# Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
# sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
# sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"

# Agregar propietarips y permisos correspondientes a los ficheros
chown -R root:root $DIR_PACKAGE/

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build $DIR_PACKAGE/
```

Modificar este template si es necesario.


#### Ejemplo 2 (usando Docker)

En el siguiente ejemplo, sera un build donde se requiera usar docker para hacer la compilación con X tecnologia y asi obtener el binario y posterior a ellos, 

**Estructura del directorio**


```bash {filename=mi-proyecto-git}
.
├── build
├── Dockerfile
└── programa
    └── hola-mundo.c

```

**Dockerfile**

Este archivo servira como base para crear una imagen base Ubuntu:20.04 con la herramientas necesaria para compilar un proyecto.
El ejemplo de muestra es de un "hola mundo" creado en C.

> Nota.- el script **build** creara la nueva imagen Docker con el tag: `plurios-builder:24.04`

```Dockerfile {filename=Dockerfile}
FROM ubuntu:24.04

RUN apt update -qq && \
    DEBIAN_FRONTEND=noninteractive apt install -y gcc && \
    apt clean && rm -rf /var/lib/apt/lists/*

WORKDIR /src

CMD ["bash"]
```

**build**

Este archivo es un poco diferente al ejemplo 1 ya que cuenta con instrucciones para utilizar docker y realizar la compilación en el con su empaquetado:


```bash {filename=build}
#!/bin/bash
set -e

# verifica si el script se ejecuta como root
if [ "$(id -u)" -ne 0 ]; then
  echo "This script must be run as root."
  exit 1
fi

# definir variables para el paquete
NAME="mi-programa"      # nombre del paquete : minusculas, usar el simbolo - en vez de espacio, sin acentos
VERSION="0.3"           # version: x.x o x.x.x (recomendable)
ARCH="amd64"            # arch: amd64, i386, all

# definir variables para docker
IMAGE_NAME="plurios-builder:24.04"
SRC_FILE="programa/hola-mundo.c"
OUTPUT="./hola-mundo"

if [[ ! -f "$SRC_FILE" ]]; then
    echo -e "\033[0;31mError: source file not found: $SRC_FILE\033[0m"
    exit 1
fi

if [[ "$(docker images -q $IMAGE_NAME 2>/dev/null)" == "" ]]; then
    echo -e "\033[0;33mBuilding Docker image '$IMAGE_NAME'...\033[0m"
    docker build -t "$IMAGE_NAME" .
fi

echo -e "\033[0;34mCompiling $SRC_FILE...\033[0m"
docker run --rm \
    -v "$PWD/programa":/src \
    -v "$PWD":/out \
    "$IMAGE_NAME" \
    bash -c "gcc /src/hola-mundo.c -o /out/hola-mundo"

echo -e "\033[0;32mBinary created: $OUTPUT\033[0m"

# Crear el nombre del directorio de trabajo 
# Ej: mi-programa_0.3_amd64
DIR_PACKAGE="${NAME}_${VERSION}_${ARCH}"

# Crea el directorio de trabajo 
mkdir -p $DIR_PACKAGE

# Copia y pega los directorios necesarios para el paquete
# DEBIAN es un directorio importante que contiene todos los metadatos de un paquete DEB
cp -r DEBIAN $DIR_PACKAGE/
cp -r usr/ $DIR_PACKAGE/                  # directorio a copiar "usr/bin/"
cp $OUTPUT $DIR_PACKAGE/usr/bin/$NAME     # copia el binario compilado con el nombre establecido para el paquete

# Calcula el tamaño en kilobytes (kB) de todo el directorio de trabajo excluyendo el directorio DEBIAN
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk '{print $1}')

# Cambia información del archivo control como ser: nombre, versión, tamaño
sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SIZE/$SIZE/g" "$DIR_PACKAGE/DEBIAN/control"

# Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
# sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
# sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"

# Agregar propietarips y permisos correspondientes a los ficheros
chown -R root:root $DIR_PACKAGE/

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build $DIR_PACKAGE/
```

Este ejemplo podra realizar la compilación del proyecto en un contenedor docker para evitar dependencias de un sistema diferente, versiones de librerias, entre otros.