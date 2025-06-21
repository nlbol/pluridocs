---
title: Contribución
type: docs
toc: true
---


Esta sección es para establecer algunos estandares importantes que debe tener un repositorio para que este pueda funcionar en DEB y que sea includio, a continuación se mostrara algunos ejemplos utiles y esenciales para que puedan reutilizarlo y asi crear su paquete deb

## Requisitos importantes

- Todo paquete debe funcionar sobre la base dePluriOS 
- El resultado (paquete deb) del proyecto no debe superar los 99MB
- Todo paquete debe ser compatible con las versiones de herramientas disponibles en el sistema o en su repositorio actual (Ubuntu 24.04 LTS) de PluriOS, ej: Python3, Perl, C/C++ y otros


### Ejemplo 1

El siguiente ejemplo es un script `build` que permite empaquetar un programa que no requiere descargar ningun recurso desde internet. Es decir, el repositorio debe tener todo lo necesario para que el paquete pueda instalarse y ejecutarse correctamente.

```bash {filename=build}
#!/bin/bash

# verificar si el script se ejecuta como root
if [ "$(id -u)" -ne 0 ]; then
  echo "This script must be run as root."
  exit 1
fi

# definir variables para el paquete
NAME="mi-programa"      # nombre del paquete
VERSION="0.3"           # version: x.x.x
ARCH="amd64"            # tipo: amd64, i386, all

# eliminar archivos no relevantes para subir a github
# estas instrucciones puede variar dependiendo que requiera el proyecto
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

# Cambia información del archivo control como ser: nombre, versión, tamaño
sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/DEBIAN/control"
sed -i "s/CHANGE_SIZE/$SIZE/g" "$DIR_PACKAGE/DEBIAN/control"

# Cambiar el nombre y versión del lanzador del programa  (si es que el programa usa un lanzador)
# sed -i "s/CHANGE_NAME/$NAME/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"
# sed -i "s/CHANGE_VERSION/$VERSION/g" "$DIR_PACKAGE/usr/share/applications/${NAME}.desktop"

# Calcula el tamaño en kilobytes (kB)
SIZE=$(du -ks --exclude=DEBIAN "$DIR_PACKAGE/" | awk '{print $1}')



chown -R root:root $DIR_PACKAGE/

dpkg-deb --build $DIR_PACKAGE/

chown -R $SUDO_USER:$SUDO_USER $DIR_PACKAGE
```