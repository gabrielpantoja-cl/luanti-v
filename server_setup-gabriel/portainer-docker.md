# Portainer: La Interfaz Gráfica para tu Servidor Docker

Este documento explica qué es Portainer, por qué es una herramienta extremadamente útil para nuestra configuración de VPS, y cómo podemos instalarlo para gestionar visualmente nuestro servidor de Luanti y las demás aplicaciones.

## 1. ¿Qué es Portainer?

Portainer es una **interfaz de usuario web, ligera y de código abierto** que te permite gestionar fácilmente tus entornos de Docker. En lugar de escribir comandos en la terminal para todo, Portainer te da un "panel de control" en tu navegador para ver y controlar tus contenedores, volúmenes, redes y más.

Piensa en ello como el "panel de administración" de tu servidor Docker.

## 2. ¿Cómo nos va a ayudar?

Para un VPS que corre múltiples aplicaciones no relacionadas (como un servidor de juegos Luanti y una aplicación web como `pitutito.cl`), Portainer es invaluable. Nos dará una visión clara y unificada de todo lo que está sucediendo en el servidor.

### Ventajas Clave para Nuestro Caso de Uso:

1.  **Visualización de Contenedores (¡Lo más importante!):**
    *   Verás una lista de todos tus contenedores (`luanti_server_production`, `pitutito-web`, todos los de Supabase, etc.).
    *   De un vistazo, sabrás su **estado** (corriendo, detenido, reiniciando), los **puertos** que usan y la **imagen** de la que provienen.
    *   Se acabó el tener que memorizar y escribir `docker ps` constantemente.

2.  **Gestión de Contenedores con un Clic:**
    *   **Start / Stop / Restart / Kill:** Puedes realizar estas acciones sobre cualquier contenedor con botones en la interfaz, lo cual es ideal para reinicios rápidos sin necesidad de acceder por SSH.
    *   **Recrear:** Portainer puede recrear un contenedor, volviendo a bajar su imagen si es necesario. Útil para forzar una actualización.

3.  **Logs en Tiempo Real en tu Navegador:**
    *   En lugar de `docker compose logs luantiserver`, puedes hacer clic en el contenedor de Luanti y ver sus logs directamente en la web. La vista se actualiza automáticamente y tiene filtros de búsqueda.
    *   Esto es increíblemente útil para depurar problemas como el que tuvimos con el servidor reiniciándose.

4.  **Inspección de Recursos (CPU y RAM):**
    *   Portainer te muestra un **gráfico en tiempo real** del uso de CPU y memoria RAM para cada contenedor. Podrás ver al instante si tu servidor de Luanti está consumiendo muchos recursos o si alguna otra aplicación tiene fugas de memoria.

5.  **Gestión de Volúmenes y Redes:**
    *   Podrás ver los volúmenes que hemos creado (como el de los mundos de Luanti) y los datos que contienen.
    *   Visualizarás las redes que Docker ha creado y qué contenedores están conectados a cada una, ayudando a entender el aislamiento entre aplicaciones.

6.  **Acceso a una Terminal dentro del Contenedor:**
    *   Si necesitas ejecutar un comando rápido dentro de un contenedor (por ejemplo, para listar archivos), Portainer te da acceso a una consola de terminal directamente desde el navegador, sin necesidad de `docker exec`.

En resumen, Portainer nos da **visibilidad y control** sobre nuestro entorno Docker, haciendo la administración del día a día mucho más simple y menos propensa a errores, especialmente al tener múltiples proyectos en el mismo VPS.

## 3. Cómo Instalar Portainer en Nuestro VPS

La belleza de Portainer es que **se instala como otro contenedor de Docker**. El proceso es muy sencillo y seguro.

### Paso 1: Crear un Volumen para los Datos de Portainer

Primero, creamos un volumen de Docker para que Portainer guarde sus propios datos de configuración. Esto asegura que no perderemos nuestros usuarios o configuración si actualizamos Portainer.

```bash
docker volume create portainer_data
```

### Paso 2: Desplegar el Contenedor de Portainer

Luego, ejecutamos un solo comando para descargar la imagen de Portainer y ponerla en marcha. Usaremos la edición "Community Edition" (CE), que es gratuita.

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

**Desglose de este comando:**
*   `-d`: Corre el contenedor en segundo plano (detached).
*   `-p 8000:8000 -p 9443:9443`: Mapea dos puertos. El `9443` es para la interfaz web segura (HTTPS) y el `8000` para HTTP.
*   `--name portainer`: Le da un nombre fácil de recordar al contenedor.
*   `--restart=always`: Asegura que Portainer se inicie siempre con el VPS.
*   `-v /var/run/docker.sock:/var/run/docker.sock`: **Esta es la parte mágica.** Mapea el "socket" de Docker del VPS al interior del contenedor. Así es como Portainer puede controlar Docker.
*   `-v portainer_data:/data`: Conecta el volumen que creamos antes para guardar los datos de Portainer.
*   `portainer/portainer-ce:latest`: La imagen oficial de Portainer Community Edition.

## 4. Primeros Pasos con Portainer

1.  **Abrir la Interfaz Web:** Una vez desplegado, abre tu navegador y ve a la IP de tu servidor en el puerto 9443:
    `https://<IP_DE_TU_VPS>:9443`
    (Tu navegador te dará una advertencia de seguridad porque el certificado es autofirmado. Es seguro aceptarla).

2.  **Crear el Usuario Administrador:** La primera vez que accedas, Portainer te pedirá que crees un usuario y una contraseña. Este será tu login de administrador.

3.  **Conectar al Entorno Local:** Portainer te preguntará qué entorno quieres gestionar. Elige la opción "Docker" y haz clic en "Connect".

¡Y ya está! Serás recibido por el panel de control principal (Dashboard) y podrás ver todos tus contenedores, incluido el propio Portainer, el servidor de Luanti y todo el stack de Supabase.
