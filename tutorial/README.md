# Tutorial guiado: Docker para desarrollo de software embebido multiplataforma

Este tutorial muestra, paso a paso, cómo preparar un entorno Docker desde cero en Ubuntu y utilizarlo para construir el mismo programa para dos arquitecturas diferentes:

- `linux/amd64`
- `linux/arm64`

El objetivo es comprobar que una computadora de desarrollo `x86_64/amd64` puede generar software destinado a `ARM64` utilizando Docker, Buildx, BuildKit y cross-compilation con Go.

---

# 1. Comprobar la arquitectura de la computadora

Abrir una terminal y ejecutar:

```bash
uname -m
```

En una computadora AMD64 se espera:

```text
x86_64
```

Esto indica que la computadora de desarrollo utiliza una arquitectura x86 de 64 bits.

Docker normalmente representa esta arquitectura como:

```text
linux/amd64
```

Más adelante utilizaremos esta computadora para generar también un ejecutable:

```text
linux/arm64
```

---

# 2. Comprobar si Docker está instalado

Ejecutar:

```bash
docker --version
```

Si Docker ya está instalado, aparecerá una versión.

Por ejemplo:

```text
Docker version ...
```

Si aparece:

```text
Command 'docker' not found
```

continuar con la instalación de Docker de la siguiente sección.

---

# 3. Instalar Docker Engine en Ubuntu

## 3.1 Actualizar la información de paquetes

Ejecutar:

```bash
sudo apt update
```

Este comando actualiza la lista de paquetes disponibles en los repositorios configurados en Ubuntu.

---

## 3.2 Instalar certificados y curl

Ejecutar:

```bash
sudo apt install ca-certificates curl
```

- `ca-certificates` permite verificar certificados HTTPS.
- `curl` se utilizará para descargar la llave del repositorio oficial de Docker.

Si aparece:

```text
Continue? [Y/n]
```

escribir:

```text
Y
```

y presionar `Enter`.

---

## 3.3 Crear el directorio para llaves de APT

Ejecutar:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Este directorio será utilizado para almacenar la llave de firma del repositorio de Docker.

Si el comando termina sin mostrar ningún mensaje, es normal.

---

## 3.4 Descargar la llave oficial de Docker

Ejecutar:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
    -o /etc/apt/keyrings/docker.asc
```

Luego:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Esto permite que APT pueda leer la llave y verificar los paquetes descargados desde Docker.

---

## 3.5 Agregar el repositorio oficial de Docker

Copiar y pegar todo este bloque:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Este archivo indica a Ubuntu que Docker debe instalarse desde el repositorio oficial de Docker.

Actualizar nuevamente:

```bash
sudo apt update
```

En la salida debería aparecer una línea relacionada con:

```text
https://download.docker.com/linux/ubuntu
```

---

## 3.6 Instalar Docker Engine, Buildx y Compose

Ejecutar:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Cuando aparezca:

```text
Continue? [Y/n]
```

escribir:

```text
Y
```

y presionar `Enter`.

Los paquetes principales que se instalan son:

- `docker-ce`: Docker Engine.
- `docker-ce-cli`: interfaz de línea de comandos de Docker.
- `containerd.io`: runtime utilizado por Docker.
- `docker-buildx-plugin`: plugin necesario para builds avanzados y multiplataforma.
- `docker-compose-plugin`: soporte para Docker Compose.

---

# 4. Verificar que Docker quedó instalado

Comprobar el servicio de Docker:

```bash
sudo systemctl status docker
```

Debe aparecer:

```text
Active: active (running)
```

Para salir de esta pantalla presionar:

```text
q
```

Ahora ejecutar:

```bash
sudo docker run hello-world
```

La primera vez Docker descargará la imagen `hello-world`.

Se espera:

```text
Hello from Docker!
```

Este comando confirma que:

1. La CLI de Docker puede comunicarse con el daemon.
2. Docker puede descargar una imagen.
3. Docker puede crear un contenedor.
4. Docker puede ejecutar el contenedor.

Comprobar Buildx:

```bash
docker buildx version
```

Debe aparecer una versión de `docker/buildx`.

---

# 5. Configurar Docker para utilizarlo sin sudo

Al ejecutar:

```bash
docker buildx ls
```

puede aparecer un error similar a:

```text
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

Esto ocurre porque el usuario actual todavía no tiene permisos para comunicarse directamente con el socket de Docker.

Agregar el usuario actual al grupo `docker`:

```bash
sudo usermod -aG docker $USER
```

La forma más sencilla de aplicar el cambio es cerrar sesión de Ubuntu y volver a entrar.

También puede intentarse:

```bash
newgrp docker
```

Luego comprobar:

```bash
groups
```

Debe aparecer:

```text
docker
```

Probar Docker nuevamente sin `sudo`:

```bash
docker run hello-world
```

---

## Si aparece `newgrp: command not found`

Instalar:

```bash
sudo apt install util-linux-extra
```

Luego:

```bash
newgrp docker
```

También se puede cerrar sesión y volver a entrar.

---

# 6. Crear la carpeta del proyecto

Crear una carpeta para el tutorial:

```bash
mkdir -p ~/docker-embedded-demo
```

Entrar:

```bash
cd ~/docker-embedded-demo
```

Comprobar la ubicación:

```bash
pwd
```

Se espera algo similar a:

```text
/home/usuario/docker-embedded-demo
```

---

# 7. Crear el programa en Go

Crear el archivo:

```bash
nano main.go
```

Pegar:

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // Mensaje principal de la aplicación.
    fmt.Println("Demo Docker para sistemas embebidos")

    // GOOS indica el sistema operativo para el cual
    // fue compilado el programa.
    fmt.Printf("Sistema operativo objetivo: %s\n", runtime.GOOS)

    // GOARCH indica la arquitectura para la cual
    // fue compilado el programa.
    fmt.Printf("Arquitectura objetivo: %s\n", runtime.GOARCH)
}
```

Guardar en `nano`:

```text
Ctrl + O
Enter
Ctrl + X
```

Comprobar el archivo:

```bash
cat main.go
```

Este programa mostrará el sistema operativo y la arquitectura para los que fue compilado.

---

# 8. Crear el Dockerfile

Crear:

```bash
nano Dockerfile
```

Pegar:

```dockerfile
# syntax=docker/dockerfile:1

# ============================================================
# ETAPA 1: COMPILACIÓN
# El compilador Go se ejecuta sobre la plataforma del host.
# ============================================================
FROM --platform=$BUILDPLATFORM golang:1.24-alpine AS build

# Plataforma donde se ejecuta el builder.
ARG BUILDPLATFORM

# Plataforma completa solicitada, por ejemplo linux/arm64.
ARG TARGETPLATFORM

# Sistema operativo objetivo.
ARG TARGETOS

# Arquitectura objetivo.
ARG TARGETARCH

# Directorio de trabajo dentro de la etapa de compilación.
WORKDIR /src

# Copiar el programa al entorno de construcción.
COPY main.go .

# Mostrar las plataformas y compilar.
#
# CGO_ENABLED=0 evita depender de un compilador C externo.
# GOOS indica el sistema operativo objetivo.
# GOARCH indica la arquitectura objetivo.
RUN echo "Build platform:  $BUILDPLATFORM" && \
    echo "Target platform: $TARGETPLATFORM" && \
    CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH \
    go build -o /out/app main.go


# ============================================================
# ETAPA 2: ARTEFACTO
# Permite extraer solamente el ejecutable generado.
# ============================================================
FROM scratch AS artifact

COPY --from=build /out/app /app


# ============================================================
# ETAPA 3: IMAGEN FINAL
# Ejemplo de una imagen que podría utilizar el ejecutable.
# ============================================================
FROM alpine:3.20 AS runtime

COPY --from=build /out/app /app

ENTRYPOINT ["/app"]
```

Guardar:

```text
Ctrl + O
Enter
Ctrl + X
```

Comprobar:

```bash
cat Dockerfile
```

---

# 9. Entender qué hace el Dockerfile

La línea:

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.24-alpine AS build
```

indica que la etapa donde se ejecuta el compilador utilizará la plataforma del Development Host.

En una computadora `x86_64`:

```text
BUILDPLATFORM = linux/amd64
```

Las variables:

```dockerfile
ARG TARGETPLATFORM
ARG TARGETOS
ARG TARGETARCH
```

son proporcionadas por BuildKit cuando se solicita un build para una plataforma determinada.

Por ejemplo, al construir con:

```text
--platform linux/arm64
```

BuildKit proporcionará valores equivalentes a:

```text
TARGETPLATFORM = linux/arm64
TARGETOS       = linux
TARGETARCH     = arm64
```

La línea:

```dockerfile
CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH \
    go build -o /out/app main.go
```

realiza la cross-compilation.

El compilador Go se ejecuta en AMD64, pero genera un ejecutable para la arquitectura especificada en `GOARCH`.

---

# 10. Comprobar los archivos creados

Ejecutar:

```bash
ls -l
```

Deben aparecer:

```text
Dockerfile
main.go
```

---

# 11. Comprobar los builders disponibles

Ejecutar:

```bash
docker buildx ls
```

Puede aparecer un builder llamado:

```text
default
```

Para este tutorial se creará un builder separado.

---

# 12. Crear el builder de Buildx

Ejecutar:

```bash
docker buildx create \
    --name embedded-builder \
    --driver docker-container \
    --use
```

- `--name embedded-builder` asigna un nombre al builder.
- `--driver docker-container` hace que BuildKit se ejecute dentro de un contenedor.
- `--use` selecciona ese builder como el activo.

Inicializarlo:

```bash
docker buildx inspect --bootstrap
```

La primera vez puede tardar porque Docker debe descargar la imagen de BuildKit.

Comprobar:

```bash
docker buildx ls
```

Se espera algo similar a:

```text
embedded-builder*       docker-container
 \_ embedded-builder0   ...   running
```

El `*` indica que ese builder está seleccionado.

---

# 13. Error posible: `context deadline exceeded`

Durante:

```bash
docker buildx inspect --bootstrap
```

puede aparecer:

```text
context deadline exceeded
```

No eliminar el builder inmediatamente.

Primero ejecutar:

```bash
docker buildx ls
```

y:

```bash
docker ps -a --filter name=buildx_buildkit_embedded-builder0
```

Si aparece:

```text
running
```

o:

```text
Up ...
```

BuildKit terminó de iniciar correctamente.

Volver a ejecutar:

```bash
docker buildx inspect embedded-builder --bootstrap
```

Si el contenedor aparece como:

```text
Exited
```

revisar los logs:

```bash
docker logs buildx_buildkit_embedded-builder0 --tail 50
```

---

# 14. Si el builder ya existe

Si al intentar crearlo nuevamente Docker indica que `embedded-builder` ya existe, no hay que crear otro.

Seleccionarlo:

```bash
docker buildx use embedded-builder
```

Inicializar:

```bash
docker buildx inspect --bootstrap
```

---

# 15. Construir primero para AMD64

Eliminar una salida anterior:

```bash
rm -rf out-amd64
```

Ejecutar:

```bash
docker buildx build \
    --platform linux/amd64 \
    --target artifact \
    --output type=local,dest=./out-amd64 \
    --progress=plain \
    .
```

Explicación de las opciones:

```text
--platform linux/amd64
```

indica que el resultado debe generarse para Linux AMD64.

```text
--target artifact
```

indica que el build debe detenerse en la etapa llamada `artifact`.

```text
--output type=local,dest=./out-amd64
```

exporta el resultado hacia la carpeta local `out-amd64`.

```text
--progress=plain
```

muestra el proceso del build de forma legible en la terminal.

Durante el build debe aparecer:

```text
Build platform:  linux/amd64
Target platform: linux/amd64
```

En este primer build ambas plataformas coinciden.

---

# 16. Verificar el binario AMD64

Ejecutar:

```bash
file out-amd64/app
```

Se espera algo similar a:

```text
ELF 64-bit LSB executable, x86-64, ...
```

La parte importante es:

```text
x86-64
```

Esto confirma que el archivo fue generado para la arquitectura AMD64/x86-64.

---

# 17. Ejecutar el binario AMD64

Ejecutar:

```bash
./out-amd64/app
```

Se espera:

```text
Demo Docker para sistemas embebidos
Sistema operativo objetivo: linux
Arquitectura objetivo: amd64
```

El programa funciona porque el host y el ejecutable utilizan una arquitectura compatible.

---

# 18. Construir el mismo programa para ARM64

No modificar `main.go`.

No modificar `Dockerfile`.

Eliminar una salida anterior:

```bash
rm -rf out-arm64
```

Ejecutar:

```bash
docker buildx build \
    --platform linux/arm64 \
    --target artifact \
    --output type=local,dest=./out-arm64 \
    --progress=plain \
    .
```

Ahora debe aparecer:

```text
Build platform:  linux/amd64
Target platform: linux/arm64
```

Aquí la plataforma del builder y la plataforma objetivo son diferentes.

Esta es la cross-compilation que se desea demostrar.

---

# 19. Verificar el binario ARM64

Ejecutar:

```bash
file out-arm64/app
```

Se espera:

```text
ELF 64-bit LSB executable, ARM aarch64, ...
```

La parte importante es:

```text
ARM aarch64
```

Esto confirma que el segundo ejecutable fue generado para ARM64.

---

# 20. Comparar ambos ejecutables

Ejecutar:

```bash
file out-amd64/app out-arm64/app
```

Se espera algo equivalente a:

```text
out-amd64/app: ELF 64-bit ... x86-64 ...
out-arm64/app: ELF 64-bit ... ARM aarch64 ...
```

Se utilizó:

- el mismo `main.go`;
- el mismo Dockerfile;
- la misma computadora;

pero se obtuvieron binarios para dos arquitecturas diferentes.

---

# 21. Intentar ejecutar el binario ARM64

Ejecutar:

```bash
./out-arm64/app
```

En una computadora x86-64 sin emulación ARM configurada se espera:

```text
bash: ./out-arm64/app: cannot execute binary file: Exec format error
```

Este error es esperado.

El ejecutable contiene instrucciones AArch64 y el procesador del host utiliza x86-64.

El hecho de que ambos sean programas para Linux no significa que sean compatibles con la misma ISA.

---

# 22. Verificación adicional con `readelf`

Para AMD64:

```bash
readelf -h out-amd64/app | grep Machine
```

Se espera:

```text
Machine: Advanced Micro Devices X86-64
```

Para ARM64:

```bash
readelf -h out-arm64/app | grep Machine
```

Se espera:

```text
Machine: AArch64
```

`readelf` inspecciona directamente la cabecera ELF del ejecutable.

---

# 23. Warning posible: `RedundantTargetPlatform`

Puede aparecer:

```text
RedundantTargetPlatform:
Setting platform to predefined $TARGETPLATFORM in FROM is redundant
```

Esto ocurre si el Dockerfile utiliza:

```dockerfile
FROM --platform=$TARGETPLATFORM alpine:3.20 AS runtime
```

Cambiarlo por:

```dockerfile
FROM alpine:3.20 AS runtime
```

El Dockerfile mostrado en este tutorial ya utiliza la versión corregida.

---

# 24. Si el build tarda mucho

La primera ejecución puede tardar porque Docker debe descargar:

- BuildKit;
- la imagen `golang:1.24-alpine`;
- el frontend del Dockerfile.

Esperar hasta que termine.

Las siguientes ejecuciones normalmente serán más rápidas porque BuildKit utiliza caché.

---

# 25. Si `file out-arm64/app` muestra `x86-64`

Revisar que el comando utilizado tenga:

```bash
--platform linux/arm64
```

También revisar la salida del build.

Debe aparecer:

```text
Target platform: linux/arm64
```

Si aparece:

```text
Target platform: linux/amd64
```

el build fue solicitado para la arquitectura equivocada.

---

# 26. Si el binario ARM64 se ejecuta correctamente en x86-64

Puede ocurrir si el sistema tiene una capa de emulación configurada.

En ese caso la comprobación principal debe hacerse con:

```bash
file out-arm64/app
```

o:

```bash
readelf -h out-arm64/app | grep Machine
```

El resultado debe seguir indicando:

```text
AArch64
```

---

# 27. Resultado final

Al finalizar deben existir:

```text
docker-embedded-demo/
├── Dockerfile
├── main.go
├── out-amd64/
│   └── app
└── out-arm64/
    └── app
```

La comparación final debe mostrar:

```bash
file out-amd64/app out-arm64/app
```

con un resultado equivalente a:

```text
out-amd64/app: ... x86-64 ...
out-arm64/app: ... ARM aarch64 ...
```

Esto demuestra que Docker y BuildKit pueden utilizarse en una computadora AMD64 para mantener el entorno de construcción y generar software destinado a una arquitectura ARM64.

