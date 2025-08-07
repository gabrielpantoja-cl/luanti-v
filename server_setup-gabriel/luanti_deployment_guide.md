# Guía de Despliegue del Servidor Luanti en VPS

Este documento detalla el proceso paso a paso para desplegar un servidor de Luanti en un VPS que ya aloja otras aplicaciones, como `pitutito.cl`.

## Paso 1: Análisis del Entorno (Completado)

- **Objetivo:** Asegurar que el nuevo servicio no entre en conflicto con los existentes.
- **Acción:** Se ejecutó `docker ps` para listar los contenedores y puertos en uso.
- **Resultado:** Se confirmó que el puerto `30000/udp`, necesario para Luanti, está libre. El stack de `pitutito.cl` y Supabase no interfiere con nuestro plan.

## Paso 2: Preparar Archivos de Configuración (Completado)

- **Objetivo:** Crear los archivos necesarios para que Docker pueda construir y ejecutar el servidor.
- **Acciones:**
    1. Se creó `docker-compose.yml` en la raíz del proyecto para orquestar el contenedor.
    2. Se creó un archivo de configuración básico `minetest.conf`.
    3. Se clonó el juego por defecto (`minetest_game`) en la carpeta `games/`.
- **Resultado:** El proyecto ahora contiene todos los archivos necesarios para el despliegue.

---

