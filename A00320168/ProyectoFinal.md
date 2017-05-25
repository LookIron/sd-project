
### Proyecto Final Distribuidos

**Universidad ICESI** 
### Estudiantes: 
**Jorge Hernandez - A00317220**

**Esteban Camacho - A00320168**

**Danielito Ocampito - A00053980**

**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:**  Despliegue automático de un espejo  
**Correo:** daniel.barragan at correo.icesi.edu.co

## Objetivos
* Emplear herramientas de aprovisionamiento automático para la realización de tareas en infraestructura
* Instalar y configurar espejos de sistemas operativos en forma automática para el soporte al aprovisionamiento de máquinas virtuales en clústers de computo
* Especificar y desplegar ambientes conformados por contenedores virtuales para la realización de tareas específicas

## Descripción
En ambientes conformados por cantidades considerables de nodos como es el caso de clústers de computo, se requiere contar con mecanismos eficientes para la actualización y despliegue de nueva infraestructura.

Para este proyecto deberá realizar el aprovisionamiento automático de un espejo por medio de las herramientas vistas en clase. Debe cumplir con los siguientes requerimientos:

* El espejo debe ser del sistema operativo ubuntu-xenial
* Debe especificar por medio de un arreglo los paquetes a instalar
* No es necesario que genere llaves nuevas cada vez que despliegue el espejo, puede crearlas previamente y precargarlas en la solución de aprovisionamiento automático a emplear
* Probar por medio del despliegue de uno o varios contenedores que el mirror ha sido correctamente desplegado

## Herramientas utilizadas
* Aptly
* Docker
* Http
* Sistema operativo ubuntu-xenial
* etc.

## Procedimiento

**2) Automatizacion de comandos necesarios para el aprovisionamiento:**

Para realizar el aprovisionamiento de los servicios solicitados (El mirror automatizado) se realizaron algunas configuraciones al archivo de configuración de Aptly (aptly.conf) el cual es ejecutado dentro del contenedor del Mirror,
al igual que su posterior instalación a traves del Dockerfile del Mirror. Dichos comandos de la instalacción de Aptly y su configuración se automatizaron a traves del Dockerfile. Los comandos automatizados fueron:

```
Reemplazar la llave que viene por defecto de otra de las llaves con las que trabaja Aptly
ADD /keys/private.asc /keys/private.asc

#Reemplazar el archivo de configuracion de Aptly	que viene preestablecido	
ADD /conf/aptly.conf /etc/aptly.conf

#Importamos la llave con privada
RUN gpg --import /keys/private.asc

#Para no dejar dicha clave expuesta se elimina
RUN rm -f /keys/private.asc

#Importar keyring
RUN gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import

#Instalamos aptly
RUN echo deb http://repo.aptly.info/ squeeze main > /etc/apt/sources.list
RUN apt-key adv --keyserver keys.gnupg.net --recv-keys 9E3E53F19C7DE460
RUN apt-get update
RUN apt-get install aptly -y

#Instalamos las dependencias que requiere Aptly			
RUN echo deb http://co.archive.ubuntu.com/ubuntu/ xenial main restricted >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get install xz-utils -y
RUN apt-get install bzip2 -y

#Agregamos el mirror a la lista del sistema
RUN echo deb http://co.archive.ubuntu.com/ubuntu/ xenial universe >> /etc/apt/sources.list
#Ejecutamos una actualizacion de librerias
RUN apt-get update
RUN apt-get install expect -y

#Seleccionar los paquetes a instalar		
RUN aptly mirror create -architectures=amd64 -filter='Priority (required) | Priority (important) | Priority (standard) | postgresql' -filter-with-deps mirror-xenial http://mirror.upb.edu.co/ubuntu/ xenial main

#Actualizar el mirror
RUN aptly mirror update mirror-xenial

```
**3) Configuracion del archivo Dockerfile del servidor Nginx:**

Para la realizacion del Mirror se utilizaron los archivos anexos en la carpeta llamada **"Solución"** la cual incluye en primer nivel el docker-compose.jml el servidor Mirror y el cliente donde dentro de cada un (Mirror, cliente) estan sus respectivas carpetas para la clave la configuracion y su Dockerfile.












