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
Section: CHANGE_SECTION
Priority: CHANGE_PRIORITY
Installed-Size: CHANGE_SIZE
Maintainer: CHANGE_MAINTAINER
Copyright: CHANGE_COPYRIGHT
Description: CHANGE_DESCRIPTION
```

> Nota.- no cambiar las variables que dicen CHANGE_* , estas variables son utilizadas en build para reemplazar la información.  



## Ejemplo 1


**Estructura de directorio**

```bash {filename=mi-proyecto-git}
.
├── build
├── DEBIAN
│   └── control
└── usr
    └── bin
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
NAME="mi-programa"      # nombre del paquete :corto, minusculas, usar el simbolo - en vez de espacio, sin acentos
VERSION="0.1"           # version: x.x o x.x.x (recomendable)
ARCH="amd64"            # arch: amd64, i386, all

# Variables para control
SECTION="misc"          # cambiar de acuerdo a su programa, caso contrario dejarlo en misc
PRIORITY="optional"     # cambiar el nivel de prioridad, aunque se recomienda opcional
MAINTAINER="User Name <email@email.com>"    # cambiar por un nombre de usuario y su correo
COPYRIGHT="GPL3"        # solo son permitidos licencias compatibles con Software Libre
DESCRIPTION=""          # agregar una descripción corta y breve del programa

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

# verifica si existe un lanzador para integrarlo con su icono en formato SVG
if [ -f $NAME.desktop ]; then
  mkdir -p $DIR_PACKAGE/{usr/share/applications/,usr/share/icons/hicolor/scalable/apps/}
  cp $NAME.desktop $DIR_PACKAGE/usr/share/applications/
  cp $NAME.svg $DIR_PACKAGE/usr/share/icons/hicolor/scalable/apps/

  # Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
  sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
  sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
fi

# Calcula el tamaño en kilobytes (kB) de todo el directorio de trabajo excluyendo el directorio DEBIAN
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk '{print $1}')

# Cambia información del archivo control como ser: nombre, versión, tamaño
sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_ARCH/$ARCH/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SECTION/$SECTION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_PRIORITY/$PRIORITY/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_MAINTAINER/$MAINTAINER/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_COPYRIGHT/$COPYRIGHT/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_DESCRIPTION/$DESCRIPTION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SIZE/$SIZE/g" "$DIR_PACKAGE/DEBIAN/control"

# Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
# sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
# sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"

# Asigna permisos de ejecución a los archivos preinst, postinst, prerm, postrm
for script in postinst preinst postrm prerm; do
  if [ -f "$DIR_PACKAGE/DEBIAN/$script" ]; then
    chmod 755 $DIR_PACKAGE/DEBIAN/$script
  fi
done

# Agregar propietarips y permisos correspondientes a los ficheros
chown -R root:root $DIR_PACKAGE/

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build $DIR_PACKAGE/
```

Modificar la plantilla si es necesario.


## Ejemplo 2 (usando Docker)

En el siguiente ejemplo, sera un script `build` donde se requiera usar docker para hacer la compilación con X tecnologia y asi obtener el binario compatible con PluriOS 

**Pre requisito**

Es necesario tener instalado docker para usar esta opción, para ello se reduce la guía en simples pasos, copie y pegue esto en su terminal:


```bash {filename=comando}
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> Guía oficial de Docker https://docs.docker.com/engine/install/ubuntu/

Tome en cuenta que esta guía asume que no tiene docker instalado y es la primera vez que esta realizando estos pasos de instalación.

**Estructura del directorio**


```bash {filename=mi-proyecto-git}
.
├── build
├── DEBIAN
│   └── control
├── Dockerfile
├── programa
│   └── hola-mundo.c
└── usr
    └── bin
```

**Dockerfile**

Este archivo servira como base para crear una imagen base Ubuntu:24.04 con las herramientas necesarias para compilar su proyecto.
El ejemplo de muestra es de un "hola mundo" creado en C.

```Dockerfile {filename=Dockerfile}
FROM ubuntu:24.04

RUN apt update -qq && \
    DEBIAN_FRONTEND=noninteractive apt install -y gcc && \
    apt clean && rm -rf /var/lib/apt/lists/*

WORKDIR /src

CMD ["bash"]
```

> Nota.- el script **build** creara la nueva imagen Docker con el tag: `plurios-builder:24.04` y este sera usado cuando se vuelva a ejecutar la compilación.

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
NAME="mi-programa"      # nombre del paquete :corto, minusculas, usar el simbolo - en vez de espacio, sin acentos
VERSION="0.1"           # version: x.x o x.x.x (recomendable)
ARCH="amd64"            # arch: amd64, i386, all

# Variables para control
SECTION="misc"          # cambiar de acuerdo a su programa, caso contrario dejarlo en misc
PRIORITY="optional"     # cambiar el nivel de prioridad, aunque se recomienda opcional
MAINTAINER="User Name <email@email.com>"    # cambiar por un nombre de usuario y su correo
COPYRIGHT="GPL3"        # solo son permitidos licencias compatibles con Software Libre
DESCRIPTION=""          # agregar una descripción corta y breve del programa

# definir variables para docker
IMAGE_NAME="plurios-builder:24.04"
SRC_DIR="programa"
OUTPUT=$NAME

if [[ ! -d "$SRC_DIR" ]]; then
    echo -e "\033[0;31mError: source folder not found: $SRC_DIR\033[0m"
    exit 1
fi

if [[ "$(docker images -q $IMAGE_NAME 2>/dev/null)" == "" ]]; then
    #echo -e "\033[0;33mBuilding Docker image '$IMAGE_NAME'...\033[0m"
    echo "Building Docker image '$IMAGE_NAME'..."
    docker build -t "$IMAGE_NAME" .
fi

echo -e "\033[1;37mCompiling $SRC_DIR...\033[0m"

docker run --rm \
    -v "$PWD/$SRC_DIR":/src \
    -v "$PWD":/out \
    "$IMAGE_NAME" \
    bash -c "gcc /src/hola-mundo.c -o /out/$OUTPUT"

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
cp $OUTPUT $DIR_PACKAGE/usr/bin/          # copia el binario compilado con el nombre establecido para el paquete

# verifica si existe un lanzador para integrarlo con su icono en formato SVG
if [ -f $NAME.desktop ]; then
  mkdir -p $DIR_PACKAGE/usr/share/applications/
  cp $NAME.desktop $DIR_PACKAGE/usr/share/applications/

  # Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
  sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
  sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
fi

# Calcula el tamaño en kilobytes (kB) de todo el directorio de trabajo excluyendo el directorio DEBIAN
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk '{print $1}')

# Cambia información del archivo control como ser: nombre, versión, tamaño
sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_ARCH/$ARCH/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SECTION/$SECTION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_PRIORITY/$PRIORITY/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_MAINTAINER/$MAINTAINER/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_COPYRIGHT/$COPYRIGHT/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_DESCRIPTION/$DESCRIPTION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SIZE/$SIZE/g" "$DIR_PACKAGE/DEBIAN/control"

# Asigna permisos de ejecución a los archivos preinst, postinst, prerm, postrm
for script in postinst preinst postrm prerm; do
  if [ -f "$DIR_PACKAGE/DEBIAN/$script" ]; then
    chmod 755 $DIR_PACKAGE/DEBIAN/$script
  fi
done

# Agregar propietarips y permisos correspondientes a los ficheros
chown -R root:root $DIR_PACKAGE/

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build $DIR_PACKAGE/
```

Este ejemplo podra realizar la compilación del proyecto en un contenedor docker para evitar dependencias de un sistema diferente, versiones de librerias, entre otros.