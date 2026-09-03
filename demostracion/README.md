# Demostración básica: ejecución de un contenedor Docker

Esta demostración introduce de forma sencilla el funcionamiento de Docker antes del tutorial guiado de compilación multiplataforma.

## Objetivo

Comprobar que Docker puede ejecutar un entorno aislado diferente al entorno de usuario del host.

La demostración utiliza únicamente `docker run`.

## 1. Requisitos previos

Antes de realizar la demostración se debe tener instalado y configurado:

- Ubuntu Linux.
- Docker Engine.
- Docker CLI.
- Acceso a Internet para descargar `alpine:3.20`, si todavía no está disponible.
- Permisos para ejecutar Docker sin `sudo`.

Comprobar Docker:

```bash
docker --version
```

Comprobar que funciona:

```bash
docker run --rm hello-world
```

Si aparece:

```text
Hello from Docker!
```

el entorno está listo.

## 2. Comprobar el sistema operativo del host

Ejecutar:

```bash
cat /etc/os-release
```

En Ubuntu se mostrará información similar a:

```text
NAME="Ubuntu"
```

Este resultado corresponde al sistema operativo del host.

## 3. Ejecutar Alpine Linux dentro de un contenedor

Ejecutar:

```bash
docker run --rm alpine:3.20 cat /etc/os-release
```

La primera vez Docker puede descargar la imagen.

Después se espera una salida similar a:

```text
NAME="Alpine Linux"
ID=alpine
```

Ahora se puede comparar:

```text
Host        -> Ubuntu
Contenedor  -> Alpine Linux
```

### ¿Qué significa el comando?

```text
docker run
```

Crea y ejecuta un contenedor.

```text
--rm
```

Elimina automáticamente el contenedor cuando termina.

```text
alpine:3.20
```

Es la imagen utilizada para crear el contenedor.

```text
cat /etc/os-release
```

Es el comando ejecutado dentro del contenedor.

## 4. Ejecutar un comando sencillo

Ejecutar:

```bash
docker run --rm alpine:3.20 echo "Hola desde un contenedor Docker"
```

Resultado esperado:

```text
Hola desde un contenedor Docker
```

Docker utiliza la imagen, crea el contenedor, ejecuta el comando y elimina el contenedor al finalizar.

## 5. Comprobar que el contenedor terminó

Ejecutar:

```bash
docker ps
```

El contenedor utilizado no debería aparecer porque ya terminó.

También se puede comprobar:

```bash
docker ps -a
```

Como se utilizó `--rm`, tampoco debería permanecer entre los contenedores detenidos.

## 6. Interpretación

La computadora continúa utilizando Ubuntu como sistema operativo del host, mientras Docker permite ejecutar un entorno Alpine Linux dentro de un contenedor.

## 7. Comandos de la demostración

```bash
cat /etc/os-release
```

```bash
docker run --rm alpine:3.20 cat /etc/os-release
```

```bash
docker run --rm alpine:3.20 echo "Hola desde un contenedor Docker"
```

```bash
docker ps
```

## 8. Errores comunes

### `docker: command not found`

Docker no está instalado. Esta demostración asume que Docker fue instalado previamente.

### `permission denied while trying to connect to the docker API`

El usuario no tiene permisos para utilizar Docker sin `sudo`.

Comprobar:

```bash
groups
```

Debe aparecer:

```text
docker
```

### Docker no puede descargar `alpine:3.20`

Comprobar la conexión a Internet.

También se puede revisar si la imagen ya está disponible localmente:

```bash
docker images
```

Debe aparecer una entrada similar a:

```text
alpine   3.20
```

## Resultado esperado

Al finalizar se habrá comprobado:

```text
Host        -> Ubuntu
Contenedor  -> Alpine Linux
```

Docker crea un contenedor temporal, ejecuta un comando dentro de él y lo elimina automáticamente al finalizar.

Esta demostración introduce el funcionamiento básico de Docker. 
