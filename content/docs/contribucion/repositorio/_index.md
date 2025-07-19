---
title: Repositorio
type: docs
toc: true
---

## Introducción 

Esta sección es para establecer algunos estándares importantes que debe tener un repositorio para que este pueda funcionar en DEB y que sea incluido.  
A continuación se mostrara algunos ejemplos útiles y esenciales para que puedan reutilizarlo y asi crear su paquete deb

## Requisitos importantes

- Todo paquete debe funcionar sobre la base de PluriOS (Ubuntu 24.04 LTS).
- El resultado (paquete deb) del proyecto no debe superar los 99MB.
- Todo paquete debe ser compatible con las versiones de herramientas disponibles en el sistema o en su repositorio actual (Ubuntu 24.04 LTS) de PluriOS, ej: Python3, Perl, C/C++ y otros.
- No se aceptaran archivos binarios (compilados/ofuscados) en el repositorio, esto como medida de seguridad para que el/los binario(s) sea(n) generado(s) desde el repositorio DEB.

## Clasificación de programas: CLI y GUI

Esta sección es para brindar información sobre la clasificación del programa que desea desarrollar o integrar.

### CLI (Command Line Interface)

Interfaz en la que el usuario interactúa con el sistema mediante comandos de texto en una terminal.

### GUI (Graphical User Interface)

Interfaz en la que el usuario interactúa con el sistema a través de elementos gráficos como ventanas, iconos y menús.

Dicho esto, si su proyecto esta en esta clasificación, entonces es necesario que en su repositorio tenga estos 2 archivos importantes dentro de un directorio llamado `extra` para que se integren:
- MAME.desktop    (Lanzador)
- NAME.svg        (Archivo logo en Formato SVG)

### Ejemplos de CLI y GUI

| Característica	| CLI (Consola)	| GUI (Gráfica) |
|-----------------|---------------| --------------|
| Interfaz	      | Texto	        | Gráfica (ventanas, botones) |
| Entrada	        | Teclado (comandos)	| Ratón + teclado |
| Nombre común	  | comando, utilidad de consola	| aplicación gráfica |
| Ejemplo	| scp, rsync, nano	| gimp, vlc, evolution |



## Plantillas

En esta sección se tendrá una muestra de los archivos "plantillas" que se puede reutilizar para crear un **paquete DEB** e información util para saber que modificar.

### MAINTAINER (Archivo)

Este archivo es importante para la construcción de paquete, ya que cuenta con variables importantes 

```bash {filename=MAINTAINER}
# =================================
# definir variables para el paquete
# =================================

# Nombre corto, en minúsculas, sin espacios ni acentos. Usa guiones: ej. mi-paquete
NAME="holamundo"

# Versión en formato semántico: x.y.z. Ej: 0.1.0, 2.0.1
VERSION="1.0.0"

# Arquitectura: amd64 (64bit), i386 (32bit), all (independiente de hardware)
ARCH="amd64"

# Sección: misc, web, utils, editors, admin, net, games, etc.
SECTION="misc"

# Prioridad: required, important, standard, optional (recomendado), extra
PRIORITY="optional"

# Nombre y correo del mantenedor del paquete
MAINTAINER="User Name <email@email.com>"

# Licencia SPDX: MIT, GPL-3.0-or-later, Apache-2.0, BSD-3-Clause, etc.
COPYRIGHT="GPL-3.0-or-later"

# Breve descripción (máx. ~80 caracteres), sin redundancia
DESCRIPTION="Hola Mundo simple en Python3"

# Variable para la descripción del lanzador
# Descripción corta del programa
COMMENT="Hola Mundo Simple para PluriOS"
```

> Nota.- Este archivo debe estar en el repositorio, no debe incluirse en el `.gitignore`

### DEBIAN (directorio)

En este directorio DEBIAN, es donde se encuentran los siguientes archivos comunes que todo paquete debe tener:
- **control:** Contiene la información principal del paquete, como el nombre, versión, descripción, dependencias, mantenedor, sección, prioridad y arquitectura.

- **Scripts de mantenimiento:** Son scripts opcionales que permiten ejecutar acciones automáticas en diferentes etapas del ciclo de vida del paquete.

- **conffiles (opcional):** Lista de archivos de configuración que no deben ser sobrescritos automáticamente durante una actualización. 

- **md5sums (opcional):** Sumas de verificación MD5 de todos los archivos instalados, utilizadas para comprobar la integridad del paquete.

### Control

Este archivo contienen toda la información de un paquete que es esencial.

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

### Scripts de mantenimiento

Estos archivos son instrucciones `bash scripting` para ejecutarse antes/después de la instalación/desinstalación de un paquete.  

- **preinst:** Script que se ejecuta **antes** de instalar el paquete.
- **postinst:** Script que se ejecuta **después** de instalar el paquete.
- **prerm:** Script que se ejecuta **antes** de eliminar el paquete.
- **postrm:** Script que se ejecuta **después** de eliminar el paquete.  


Esta es la estructura de un script

```bash {filename=script}
#!/bin/bash

instrucción bash 1
instrucción bash 2
instrucción bash 3

```

Algunos ejemplos de instrucciones que puede ser usado:  
- habilitar servicios con systemd por defecto.
- crear enlaces de binarios.
- preparar los requerimientos o configuraciones necesarias antes de instalar el paquete.
- revertir o remover las acciones mencionadas cuando se elimina el paquete.
- etc..


### Permisos

Estos archivos mencionados `preinst, postinst, prerm, postrm` deben tener el siguiente permiso utilizando `chmod` para que pueda funcionar correctamente dentro del paquete DEB.  

```comando {filename=comando}
chmod 755
```

> **Nota.-** El script `build` ya tiene incluido estos pasos para evitar que haya problemas en el proceso del empaquetado.

### Licencia

En el siguiente enlace puede encontrar una lista de Licencias que puede asignar en el paquete
https://spdx.org/licenses/ y agregar el Identificador **(Identifier)** en el script **build** para que este pueda añadirlo en el archivo `control` , algunos ejemplos de licencia:

|  Identifier  |
|:------------:|
|GPL-3.0-only|
|GPL-3.0-or-later|
|GPL-2.0-only|

> **Nota.-** Use la licencia que sea compatible con Software Libre.

### lanzador

Esta plantilla solo es util si su programa tiene o utiliza interfaz gráfica (GUI).  


```bash {filename=NAME.desktop}
Name=CHANGE_NAME
Comment=CHANGE_COMMENT
Version=CHANGE_VERSION
Icon=CHANGE_NAME
Exec=CHANGE_NAME
Terminal=false
Type=Application
Categories=PluriOS;
```
> **Nota.-** Se recomienda no cambiar estos valores para que los programas estén en la categoría de PluriOS.

El script `build` detecta automáticamente si hay un lanzador creado y si hay un icono en formato SVG para que sea incluido, si esta condición no se cumple, entonces el paquete se creara sin un lanzador. 
### Versionado de Software

La siguiente información sera de utilidad para saber como asignar una versión para el empaquetado.

- **mayor:** el software sufre grandes cambios y mejoras. Ej: versión 4.0 a versión 5.0
- **menor:** el software sufre pequeños cambios y/o correcciones de errores. Ej: versión 4.1 a versión 4.2
- **micro:** se aplica una corrección al software, y a su vez sufre pocos cambios. Ej: versión 3.1.2 a versión 3.1.3

## Ejemplo 1

Para este ejemplo, se hace referencia de que el proyecto es simplemente un archivo escrito en un lenguaje como python que no requiera instrucciones de compilación y tampoco requiera el uso de librerías propias.  
En caso desarrollar librerías propias, crear su directorio correspondiente en `/usr/share/${NAME}/` .  

**Estructura de directorio**

```bash {filename=mi-proyecto-git}
.
├── build
├── DEBIAN
│   └── control
├── holamundo    <- código python
└── usr
    └── bin
```

**archivo holamundo**

el siguiente código es solo un demo de ejemplo usado para em empaquetamiento

```python {filename=holamundo}
#!/bin/python3

print("hola mundo")
```

**build**

El siguiente ejemplo es un script `build` que permite empaquetar un programa que no requiere descargar ningún recurso desde internet. Es decir, el repositorio debe tener todo lo necesario para que el paquete pueda instalarse y ejecutarse correctamente.

```bash {filename=build}
#!/bin/bash

# verifica si el script se ejecuta como root
if [ "$(id -u)" -ne 0 ]; then
  echo "This script must be run as root."
  exit 1
fi

# importar variables MAINTAINER
. MAINTAINER || exit 1


# eliminar archivos no relevantes para subir a github
# estas instrucciones puede variar dependiendo los archivos innecesarios del proyecto
rm -rf *.deb *.tar.gz ${NAME}_${ARCH}*  2>/dev/null

# Crear el nombre del directorio de trabajo 
# Ej: mi-programa_0.3_amd64
DIR_PACKAGE="${NAME}_${VERSION}_${ARCH}"

# Crea el directorio de trabajo 
mkdir -p $DIR_PACKAGE 

# Crea el directorio DEBIAN
mkdir -p  $DIR_PACKAGE/DEBIAN/

# Genera el archivo control de DEBIAN
cat <<EOF > $DIR_PACKAGE/DEBIAN/control
Package: CHANGE_NAME
Version: CHANGE_VERSION
Architecture: CHANGE_ARCH
Section: CHANGE_SECTION
Priority: CHANGE_PRIORITY
Installed-Size: CHANGE_SIZE
Maintainer: CHANGE_MAINTAINER
Copyright: CHANGE_COPYRIGHT
Description: CHANGE_DESCRIPTION
EOF

# ================================
# Copia y pega los directorios necesarios para el paquete
# modifique esta sección para copiar los archivos necesarios de su código al directorio de trabajo $DIR_PACKAGE
# ================================

mkdir -p $DIR_PACKAGE/usr/bin/

cp $NAME $DIR_PACKAGE/usr/bin/

chmod +x $NAME

mv $DIR_PACKAGE/usr/bin/


# ================================

# verifica si existe un lanzador para integrarlo con su icono en formato SVG
if [ -f "$NAME.desktop" ] && [ -f "$NAME.svg" ]; then
  mkdir -p $DIR_PACKAGE/{usr/share/applications/,usr/share/icons/hicolor/scalable/apps/}
  cp $NAME.desktop $DIR_PACKAGE/usr/share/applications/
  cp $NAME.svg $DIR_PACKAGE/usr/share/icons/hicolor/scalable/apps/
  
  # Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
  sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
  sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
  sed -i "s/CHANGE_COMMENT/$COMMENT/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
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

# Agrega propietarios y permisos correspondientes a los ficheros
chown -R root:root $DIR_PACKAGE/

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build $DIR_PACKAGE/
```

Modificar la plantilla si es necesario.

Como resultado, se obtendrá un paquete deb que puede verificar su información utilizando

```bash {filename=bash}
sudo apt info ./holamundo*.deb
Package: holamundo
Version: 1.0.0
Priority: optional
Section: misc
Maintainer: User Name <email@email.com>
Installed-Size: 16,4 kB
Copyright: GPL3
Download-Size: 828 B
APT-Sources: /home/user/mi-proyecto-git/holamundo_1.0.0_amd64.deb
Description: Hola Mundo en Python3
```


## Ejemplo 2 (usando Docker)

En el siguiente ejemplo, sera un script `build` donde se requiera usar docker para hacer la compilación con X tecnología y asi obtener el binario compatible con PluriOS 

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
├── extra
│   ├── holamundo.desktop
│   └── holamundo.svg
└── src
    └── holamundo.c
```

**Dockerfile**

Este archivo servirá como base para crear una imagen base Ubuntu:24.04 con las herramientas necesarias para compilar su proyecto.
El ejemplo de muestra es de un "hola mundo" creado en C.

```Dockerfile {filename=Dockerfile}
FROM ubuntu:24.04

RUN apt update -qq && \
    DEBIAN_FRONTEND=noninteractive apt install -y gcc && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /src

CMD ["bash"]
```

> Nota.- el script **build** creara la nueva imagen Docker con el tag: `plurios-builder-${NAME}:24.04` y este sera usado cuando se vuelva a ejecutar la compilación.

**hola-mundo.c**  

El siguiente código escrito en C, es un ejemplo que sirve para demostración de compilación usando docker 

```c {filename=hola-mundo.c}
#include <stdio.h>

int main() {
    printf("Hola, mundo desde C en Docker!\n");
    return 0;
}
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

# =================================
# definir variables para el paquete
# =================================
NAME="holamundo"        # Nombre corto, en minúsculas, sin espacios ni acentos. Usa guiones: ej. mi-paquete
VERSION="1.0.0"         # Versión en formato semántico: x.y.z. Ej: 0.1.0, 2.0.1
ARCH="amd64"            # Arquitectura: amd64 (64bit), i386 (32bit), all (independiente de hardware)
SECTION="misc"          # Sección: misc, web, utils, editors, admin, net, games, etc.
PRIORITY="optional"     # Prioridad: required, important, standard, optional (recomendado), extra
MAINTAINER="User Name <email@email.com>"  # Nombre y correo del mantenedor del paquete
COPYRIGHT="GPL-3.0-or-later"  # Licencia SPDX: MIT, GPL-3.0-or-later, Apache-2.0, BSD-3-Clause, etc.
DESCRIPTION="Hola Mundo simple en Python3"  # Breve descripción (máx. ~80 caracteres), sin redundancia

# Variable para la descripción del lanzador
COMMENT="Hola Mundo Simple para PluriOS"   # Descripción corta del programa
# =================================

# definir variables para docker
IMAGE_NAME="plurios-builder-${NAME}:24.04"
SRC_DIR="src"
DEBIAN_DIR="DEBIAN"
OUTPUT_DIR="$(pwd)"
EXTRA_DIR="extra"
EXTRA_ARGS=()

if [[ ! -d "$SRC_DIR" ]]; then
    echo -e "\033[0;31mError: source folder not found: $SRC_DIR\033[0m"
    exit 1
fi

if [[ ! -d "$DEBIAN_DIR" ]]; then
    echo -e "\033[0;31mError: no se encontró el directorio:  $DEBIAN_DIR\033[0m"
    exit 1
fi

if [[ -d "$EXTRA_DIR" ]]; then
    echo -e "\033[1;34mSe encontro el directorio extra, se empaquetarán también.\033[0m"
	EXTRA_ARGS+=(-v "$PWD/extra":/extra)
fi

if [[ "$(docker images -q $IMAGE_NAME 2>/dev/null)" == "" ]]; then
    echo "Building Docker image '$IMAGE_NAME'..."
    docker build -t "$IMAGE_NAME" .
fi

docker run --rm \
    -e NAME="$NAME" \
    -e VERSION="$VERSION" \
    -e ARCH="$ARCH" \
    -e SECTION="$SECTION" \
    -e PRIORITY="$PRIORITY" \
    -e MAINTAINER="$MAINTAINER" \
    -e COPYRIGHT="$COPYRIGHT" \
    -e DESCRIPTION="$DESCRIPTION" \
    -e COMMENT="$COMMENT" \
    -v "$PWD/$SRC_DIR":/src \
    -v "$PWD/$DEBIAN_DIR":/debian \
    -v "$OUTPUT_DIR":/out \
    "${EXTRA_ARGS[@]}" \
    "$IMAGE_NAME" bash -c '
set -e
cd /tmp

DIR_PACKAGE="${NAME}_${VERSION}_${ARCH}"
mkdir -p "$DIR_PACKAGE/usr/bin" "$DIR_PACKAGE/DEBIAN"

echo "Compilando /src/${NAME}.c → /usr/bin/${NAME} ..."
gcc /src/${NAME}.c -o "$DIR_PACKAGE/usr/bin/$NAME"

echo "Copiando DEBIAN/ ..."
cp -r /debian/* "$DIR_PACKAGE/DEBIAN/"

# verifica si existe un lanzador para integrarlo con su icono en formato SVG
if [ -f "/extra/${NAME}.desktop" ] && [ -f "/extra/${NAME}.svg" ]; then
    echo "Copiando .desktop y .svg al paquete ..."
    mkdir -p "$DIR_PACKAGE/usr/share/applications" "$DIR_PACKAGE/usr/share/icons/hicolor/scalable/apps"
    cp "/extra/${NAME}.desktop" "$DIR_PACKAGE/usr/share/applications/"
    cp "/extra/${NAME}.svg" "$DIR_PACKAGE/usr/share/icons/hicolor/scalable/apps/"

  # Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
    sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
    sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
    sed -i "s/CHANGE_COMMENT/$COMMENT/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
fi

# Calcula el tamaño en kilobytes (kB) de todo el directorio de trabajo excluyendo el directorio DEBIAN
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk "{print \$1}")

# Cambia información del archivo control como ser: nombre, versión, tamaño
echo "Actualizando control ..."
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
    chmod 755 "$DIR_PACKAGE/DEBIAN/$script"
  fi
done

# Agrega propietarios y permisos correspondientes a los ficheros
chown -R root:root "$DIR_PACKAGE/"

# Crea el paquete DEB a partir del directorio de trabajo
dpkg-deb --build "$DIR_PACKAGE" "/out/${DIR_PACKAGE}.deb"

echo -e "\033[0;32mDEB creado en ./${DIR_PACKAGE}.deb\033[0m"
'