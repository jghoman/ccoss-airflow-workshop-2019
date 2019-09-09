# Dev-environment para Airflow

Vamos a seguir la guia oficial de Airflow para [contribuir](https://github.com/apache/airflow/blob/master/CONTRIBUTING.md) , Usando [Airflow Breeze](https://github.com/apache/airflow/blob/master/BREEZE.rst) y sus comandos para el desarrollo.

## Pre-requisitos para todas las plataformas

- Git & una cuenta de GitHub
  - Vamos a usar GitHub para abrir Pull Requests de nuestras contribuciones
  - Para esto, asegurate de tener instalado Git en tu computadora, y que tu cuenta personal de GitHub este vinculada localmente. Esto, para poder clonar repositorios y compartir nuestro codigo.

- Clonar / Fork Airflow:
  - Ve al repositorio de Airflow https://github.com/apache/airflow y haz click en el icono de `Fork` . Esto va a hacer un `Fork` de Airflow en tu propia cuenta, que es lo que necesitamos para mandar Pull Requests y compartir nuestro codigo. 
  - En tu computadora, clona este nuevo repositorio via `git clone git@github.com:<tu_usuario>/airflow.git` or `git clone https://github.com/<tu_usuario>/airflow.git`.


- Docker & Docker Compose
    - Sigue las instrucciones para tu plataforma de [Docker](https://docs.docker.com/install/) y de [Docker-Compose](https://docs.docker.com/compose/install/) para instalarlos.
    - Si estas en macOS, por favor sigue estas intrucciones para aumentar el espacio para Docker https://docs.docker.com/docker-for-mac/space/

## macOS / Unix / Linux Setup

En estas plataformas necesitamos instalar dos paquetes mas: gnu getopt y gstat.
Para macOS, si tienes homebrew, los puedes intalar por homebrew via `brew install gnu-getopt coreutils`.
Terminando la instalacion, agregar el folder a tu PATH via:
```
echo 'export PATH="/usr/local/opt/gnu-getopt/bin:$PATH"' >> ~ .bash_profile
. ~/.bash_profile
```

Para Linux, puedes instalarlos via `apt install util-linux coreutils` en Debian (o equivalente en otras distros). No se olviden de tambien agregar el folder a su PATH.
 
Antes de usar Airflow Breeze, asegurate de que Docker este corriendo. En macOS puedes hacerlo con solo abrir la app de Docker desde Applications.

Ya habiendo clonado Airflow, ve a ese folder (usualmente `cd airflow`), y corre `./breeze`. Esto va a inicializar el dev-env de Airflow breeze. Este comando va a descargar unas imagenes de Docker y hacer una configuracion inicial - asi que la primera vez tomara un poco de tiempo.

Ya finalizado `breeze` , tendras todos los componentes listos para empezar tus contribuciones, y te dejara en una terminal dentro del contenedor de Airflow.

## Windows
