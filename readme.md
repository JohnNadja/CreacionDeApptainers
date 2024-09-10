# Introducción a Apptainer en Ubuntu 22.04

Apptainer, anteriormente conocido como Singularity, es una solución de contenedores diseñada específicamente para facilitar la creación, ejecución y gestión de contenedores en entornos de computación de alto rendimiento (HPC) y ciencia de datos. A diferencia de otras soluciones de contenedores, Apptainer permite a los usuarios empaquetar aplicaciones, bibliotecas y sus dependencias en un contenedor de forma que pueda ejecutarse de manera eficiente y segura en cualquier entorno Linux, incluido Ubuntu 22.04.

La creación de contenedores en Ubuntu 22.04 con Apptainer es sencilla y directa. Los usuarios pueden comenzar instalando Apptainer desde los repositorios oficiales o mediante la compilación del código fuente. Una vez instalado, se pueden crear contenedores a partir de definiciones escritas en archivos de texto, lo que permite una reproducibilidad y portabilidad excepcionales de las aplicaciones científicas y de investigación entre diferentes plataformas y entornos de computación.

Este documento proporcionará una guía paso a paso sobre cómo instalar Apptainer en Ubuntu 22.04  desde los repositorios oficiales (o mediante la compilación del código fuente) y cómo comenzar a crear y gestionar contenedores.


# índice

- [Requisitos previos](#requisitos-previos)
- [Paquetes de software necesarios](#paquetes-de-software-necesarios)
- [Instalación de Go](#instalación-de-go)
- [Paso 1: Instalar Apptainer en Ubuntu 22.04](#paso-1-instalar-apptainer-en-ubuntu-2204)
- [Paso 2: Crear un contenedor con Apptainer](#paso-2-crear-un-contenedor-con-apptainer)
- [Paso 3: Crear archivos de definición de contenedor](#paso-3-crear-archivos-de-definición-de-contenedor)


## Requisitos previos

Antes de comenzar con esta guía, asegúrese de tener lo siguiente:

- Una máquina Ubuntu 22.04 con acceso a Internet.
- Un usuario no root con privilegios sudo.
- Conocimientos básicos de la línea de comandos de Linux.


## Paquetes de software necesarios

Para instalar Apptainer en Ubuntu 22.04, primero debe instalar algunos paquetes de software necesarios. Puede hacerlo ejecutando el siguiente comando:

```bash
sudo apt update

sudo apt-get install -y build-essential libseccomp-dev pkg-config uidmap squashfs-tools fakeroot cryptsetup tzdata dh-apparmor curl wget git iproute2 net-tools
```

### Instalación de Go

Apptainer está escrito en Go, por lo que necesitará instalar Go en su sistema. Puede hacerlo siguiendo estos pasos:

1. Descargue la última versión de Go desde el sitio web oficial de Go. En el momento de escribir este artículo, la última versión estable es 1.17.1. Puede verificar la última versión en el [sitio web oficial de Go](https://golang.org/dl/).

```bash
export VERSION=1.20.10 OS=linux ARCH=amd64 

wget -O /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz https://dl.google.com/go/go${VERSION}.${OS}-${ARCH}.tar.gz && \
sudo tar -C /usr/local -xzf /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz
```

2. Configure las variables de entorno de Go. Para hacerlo, agregue las siguientes líneas al final de su archivo `~/.bashrc`:
```bash	
echo 'export GOPATH=${HOME}/go' >> ~/.bashrc && \
echo 'export PATH=/usr/local/go/bin:${PATH}:${GOPATH}/bin' >> ~/.bashrc && \
source ~/.bashrc
```





3. Instalar golangci-lint
```bash
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.59.1
```


4. Añaadir la variable de entorno de golangci-lint
```bash
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

5. Verifique la instalación de Go ejecutando el siguiente comando:

```bash
go version
```

Debería ver una salida similar a la siguiente:

```plaintext
go version go1.20.10 linux/amd64
```

6. Adicionalmente, cree el directorio de trabajo de Go y configure la variable de entorno `GOPATH`:

```bash
mkdir -p ${GOPATH}/src/github.com/sylabs 
```



## Paso 1: Instalar Apptainer en Ubuntu 22.04

En esta configuración, instalaremos Apptainer desde los repositorios oficiales de Ubuntu 22.04. Para hacerlo, siga estos pasos:

Para nuestra preferencia, estaremos instalando en `usr/local` para que todos los usuarios puedan acceder a Apptainer. Si solo desea instalarlo para el usuario actual, puede cambiar el directorio de instalación a algún directorio en su directorio de inicio.

1. Clone el repositorio de Apptainer desde GitHub:

```bash
git clone https://github.com/apptainer/apptainer.git
cd apptainer
```

2. Configurar la rama de desarrollo:

```bash
git checkout v1.3.2
```

Esto le permitirá instalar la versión estable más reciente de Apptainer.

3. Compile e instale Apptainer en su sistema:

```bash
./mconfig
cd $(/bin/pwd)/builddir
make
sudo make install
```

Si se desea ejecutar con `--with-suid` (permite que los contenedores se ejecuten con privilegios de root):

```bash
./mconfig --with-suid
cd $(/bin/pwd)/builddir
make
sudo make install
```


4. Verifique la instalación de Apptainer ejecutando el siguiente comando:

```bash
apptainer --version
```

Debería ver una salida similar a la siguiente:

```plaintext
apptainer version 1.3.2
```

Con esto, ha instalado Apptainer en su sistema Ubuntu 22.04.

## Paso 2: Crear un contenedor con Apptainer

Una vez que haya instalado Apptainer en su sistema, puede comenzar a crear contenedores. Para crear un contenedor con Apptainer, siga estos pasos:

1. Cree un archivo de definición de contenedor. Por ejemplo, cree un archivo llamado `mycontainer.def` con el siguiente contenido:

```plaintext
Bootstrap: docker
From: ubuntu:22.04 

%post
    apt-get update
    apt-get install -y python3
```

Este archivo de definición de contenedor instalará Python 3 en un contenedor Ubuntu 22.04.

2. Cree el contenedor a partir del archivo de definición ejecutando el siguiente comando:

```bash
sudo apptainer build mycontainer.def
```

Esto creará un contenedor basado en Ubuntu 22.04 con Python 3 instalado.

3. Ejecute el contenedor recién creado con el siguiente comando:

```bash
sudo apptainer run mycontainer.sif python3 --version
```

Debería ver una salida similar a la siguiente:

```plaintext
Python 3.10.0
```




Con esto, ha creado y ejecutado un contenedor con Apptainer en Ubuntu 22.04.

## Conclusión

En este documento, hemos explorado cómo instalar Apptainer en Ubuntu 22.04 desde los repositorios oficiales y cómo crear y ejecutar contenedores con Apptainer. Apptainer es una herramienta poderosa que facilita la creación, ejecución y gestión de contenedores en entornos de computación de alto rendimiento y ciencia de datos. Con Apptainer, los usuarios pueden empaquetar aplicaciones, bibliotecas y sus dependencias en contenedores para una ejecución eficiente y segura en cualquier entorno Linux.

Para obtener más información sobre Apptainer, consulte la [documentación oficial de Apptainer](https://apptainer.org/docs/).



### [Creación de InfluxDB con Telegraf](./BD_influx/Influx_%26_Telegraf.md)

### [Conexión de InfluxDB a Grafana](./BD_influx/Grafana_%26_Influx.md)




