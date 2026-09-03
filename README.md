# Mini-taller Docker en Sistemas Embebidos

Repositorio del mini-taller **“Desarrollo de software embebido a partir de contenedores: Caso Docker”**, desarrollado para el curso **Taller de Sistemas Embebidos** del Instituto Tecnológico de Costa Rica.

## Objetivo

Mostrar cómo Docker puede utilizarse en el desarrollo de software embebido para:

- crear entornos de ejecución y desarrollo aislados;
- mantener herramientas y dependencias dentro de contenedores;
- construir software para arquitecturas diferentes a la del host;
- comprender la diferencia entre usar Docker en el **Development Host** y ejecutar contenedores directamente en un **Embedded Target**.

## Contenido del repositorio

### Presentación

En [`presentacion/`](./presentacion/) se encuentra la presentación utilizada para la parte teórica del mini-taller.

Los temas principales son:

- contenedores vs. máquinas virtuales;
- Dockerfile, imágenes y contenedores;
- namespaces y cgroups;
- Development Host vs. Embedded Target;
- AMD64 vs. ARM64;
- imágenes multi-platform;
- QEMU, nodos nativos y cross-compilation;
- BuildKit y multi-stage builds;
- casos de estudio y overhead.

### Demostración

En [`demostracion/`](./demostracion/) se encuentra una demostración básica del funcionamiento de Docker.

La actividad compara el sistema operativo del host con el entorno de usuario dentro de un contenedor Alpine Linux.

Ejemplo:

```bash
cat /etc/os-release
```

Luego:

```bash
docker run --rm alpine:3.20 cat /etc/os-release
```

Esto permite observar que el **host puede utilizar Ubuntu**, mientras el contenedor utiliza un entorno Alpine Linux.

También se ejecuta un comando sencillo dentro del contenedor:

```bash
docker run --rm alpine:3.20 echo "Hola desde un contenedor Docker"
```

La demostración introduce de forma simple los conceptos de:

- imagen;
- contenedor;
- ejecución aislada;
- uso de `docker run`;
- eliminación automática del contenedor mediante `--rm`.

### Tutorial guiado

En [`tutorial/`](./tutorial/) se encuentra el ejercicio práctico que realizan los participantes.

El tutorial comienza desde la preparación del entorno e incluye:

- instalación y verificación de Docker;
- configuración de permisos;
- creación de una aplicación en Go;
- creación del Dockerfile;
- configuración de Docker Buildx y BuildKit;
- compilación para `linux/amd64`;
- cross-compilation para `linux/arm64`;
- verificación de los binarios generados;
- troubleshooting de errores comunes.

El flujo principal del tutorial es:

```text
Development Host
linux/amd64
      |
      v
Docker + BuildKit
      |
      v
Cross-compilation
      |
      +-------------------+
      |                   |
      v                   v
Binario AMD64         Binario ARM64
```

La arquitectura de los ejecutables se verifica con:

```bash
file out-amd64/app out-arm64/app
```

Resultado esperado:

```text
out-amd64/app: ... x86-64 ...
out-arm64/app: ... ARM aarch64 ...
```

### Referencias

Las fuentes técnicas y académicas utilizadas se encuentran en [`recursos/referencias.md`](./recursos/referencias.md).

## Referencias principales

- Docker Documentation — Docker Overview  
  https://docs.docker.com/get-started/docker-overview/

- Docker Documentation — Dockerfile Reference  
  https://docs.docker.com/reference/dockerfile/

- Docker Documentation — Multi-platform Builds  
  https://docs.docker.com/build/building/multi-platform/

- Yocto Project Documentation — Development Environment  
  https://docs.yoctoproject.org/dev/overview-manual/development-environment.html

- Open Container Initiative — Runtime Specification  
  https://github.com/opencontainers/runtime-spec

- L. Wen et al., “Cloud-Native Fog Robotics: Model-Based Deployment and Evaluation of Real-Time Applications,” *IEEE Robotics and Automation Letters*, vol. 10, no. 1, pp. 398–405, 2025. DOI: `10.1109/LRA.2024.3504243`

- F. Pan et al., “Toward Software-Defined Vehicles: From Model-Based Engineering to Virtualization-Based Deployment,” *IEEE Access*, vol. 12, pp. 192127–192145, 2024. DOI: `10.1109/ACCESS.2024.3512002`

## Autor

**Verónica Cambronero Solano**  
Instituto Tecnológico de Costa Rica  
Taller de Sistemas Embebidos — 2026
