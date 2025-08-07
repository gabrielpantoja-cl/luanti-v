# Guía de Docker para el Proyecto

Este documento explica cómo utilizar Docker para crear un entorno de desarrollo y producción consistente y portable para la aplicación.

## Requisitos

-   Tener instalado [Docker](https://docs.docker.com/get-docker/)
-   Tener instalado [Docker Compose](https://docs.docker.com/compose/install/)

## Archivos Clave

-   `Dockerfile`: Es la "receta" para construir la imagen de nuestra aplicación. Define el entorno, las dependencias y los comandos necesarios para que la aplicación se ejecute.
-   `docker-compose.yml`: Permite definir y orquestar múltiples servicios. En nuestro caso, define el servicio `pitutito-web` basado en nuestro `Dockerfile`.
-   `nginx.conf`: Archivo de configuración para el servidor web Nginx, que se encarga de servir los archivos de la aplicación construida.
-   `.dockerignore`: Similar a `.gitignore`, especifica qué archivos y directorios deben ser ignorados por el contexto de Docker al construir la imagen, para evitar enviar archivos innecesarios.

## Cómo Construir y Ejecutar la Aplicación

1.  **Construir la imagen:**

    ```bash
    docker-compose build
    ```

2.  **Iniciar el contenedor:**

    ```bash
    docker-compose up -d
    ```

    La aplicación estará disponible en `http://localhost:8080`.

3.  **Ver los logs:**

    ```bash
    docker-compose logs -f
    ```

4.  **Detener el contenedor:**

    ```bash
    docker-compose down
    ```

## Explicación del `Dockerfile`

El `Dockerfile` utiliza una técnica llamada **multi-stage build** para optimizar la imagen final.

-   **Etapa `build`:** Se utiliza una imagen completa de Node.js para instalar todas las dependencias de desarrollo y construir la aplicación (`bun run build`). Esto genera una carpeta `dist` con los archivos estáticos optimizados.
-   **Etapa final:** Se utiliza una imagen muy ligera de Nginx. Solo se copian los archivos de la carpeta `dist` de la etapa anterior. El resultado es una imagen final muy pequeña, sin las dependencias de desarrollo, lo que la hace más segura y rápida.
