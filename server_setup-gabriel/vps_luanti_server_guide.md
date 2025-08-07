# Guía de Despliegue de Servidor Luanti en un VPS

Esta guía adapta las prácticas de despliegue profesionales vistas en "otro-proyecto" (pitutito.cl) para la configuración de un servidor de Luanti robusto, seguro y escalable en un VPS (como DigitalOcean).

## 1. Filosofía y Arquitectura

Adoptaremos los siguientes principios clave:

*   **Todo en Docker:** El servidor de Luanti, y cualquier servicio de apoyo, se ejecutarán en contenedores Docker. Esto asegura consistencia, aislamiento y portabilidad.
*   **Infraestructura como Código (IaC):** La configuración del servidor se definirá en archivos (`docker-compose.yml`, scripts de setup), haciendo el proceso repetible y fácil de versionar.
*   **Seguridad por Defecto:** Configuraremos un firewall, certificados SSL y seguiremos las mejores prácticas para proteger el servidor desde el inicio.
*   **Escalabilidad Planificada:** Empezaremos con una configuración sencilla y económica, pero con un claro camino para escalar si tu servidor crece en popularidad.

### Arquitectura Propuesta para Luanti

```
🌍 Internet (Puerto 30000/udp)
     ↓
🔒 Firewall (UFW) en el VPS
     ↓
🐳 Docker Engine
     ├─ Contenedor: Servidor Luanti
     │  ├─ bin/luantiserver (el ejecutable)
     │  ├─ /var/lib/minetest/.minetest/ (datos del mundo, mods, etc.)
     │  └─ /etc/minetest/minetest.conf (configuración)
     │
     └─ (Opcional) Contenedor: Nginx (para web admin o stats)
     └─ (Opcional) Contenedor: Prometheus (para métricas)
```

## 2. Configuración del VPS (DigitalOcean)

### Elección del Droplet (Servidor)

Basado en el plan de "otro-proyecto", esta es una excelente estrategia:

*   **Fase 1 (Inicial):**
    *   **Plan:** Droplet Básico Premium AMD/Intel
    *   **CPU/RAM:** 1 vCPU / 2GB RAM (o 2vCPU / 2GB RAM si el presupuesto lo permite)
    *   **Costo Aproximado:** $14-21/mes
    *   **Suficiente para:** Un servidor Luanti con un número moderado de jugadores y mods.

*   **Fase 2 (Crecimiento):**
    *   **Plan:** Droplet Básico Premium AMD/Intel
    *   **CPU/RAM:** 2 vCPU / 4GB RAM
    *   **Costo Aproximado:** $28-42/mes
    *   **Cuándo escalar:** Si notas lag (retraso) constante, el uso de CPU se mantiene sobre 80%, o si quieres añadir más servidores o servicios pesados.

### Configuración Inicial del VPS

Estos pasos solo se hacen una vez, usando los scripts de `otro-proyecto` como inspiración.

1.  **Crear el Droplet:** Usa Ubuntu 22.04 LTS.
2.  **Configurar Acceso SSH:** Usa claves SSH, no contraseñas.
3.  **Configurar Firewall (UFW):**
    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow ssh       # Puerto 22 para administrar el servidor
    sudo ufw allow 30000/udp # Puerto por defecto de Luanti
    sudo ufw allow 30000/tcp # Para el server list y pings
    sudo ufw enable
    ```
4.  **Instalar Docker y Docker Compose:**
    ```bash
    # Instalar Docker
    sudo apt-get update
    sudo apt-get install -y docker.io
    # Instalar Docker Compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    # Añadir tu usuario al grupo de docker para no usar 'sudo'
    sudo usermod -aG docker $USER
    newgrp docker # Aplica los cambios en la sesión actual
    ```

## 3. Dockerizando el Servidor Luanti

Usaremos `docker-compose` para gestionar nuestro servidor. Esto hace que iniciar, parar y configurar sea trivial.

### Estructura de Archivos en el Servidor

Clonaremos nuestro fork de Luanti en el VPS. Dentro, crearemos nuestra carpeta `server_setup-gabriel`.

```
/home/gabriel/Luanti/
├── Dockerfile
├── docker-compose.yml  <-- ¡Lo crearemos ahora!
├── games/
├── worlds/
├── mods/
└── server_setup-gabriel/
    └── ... (nuestros archivos de configuración)
```

### Creando el `docker-compose.yml`

Este archivo orquesta la construcción y ejecución del servidor. Lo crearemos en la raíz de nuestro proyecto Luanti.

**`docker-compose.yml`:**
```yaml
version: '3.8'

services:
  luantiserver:
    # Construye la imagen usando el Dockerfile del repositorio
    build: .
    container_name: luanti_server_1
    restart: unless-stopped

    # Mapea el puerto del juego
    ports:
      - "30000:30000/udp"
      - "30000:30000/tcp"

    # Monta los volúmenes para persistir los datos
    volumes:
      # Mundo del juego
      - ./worlds:/var/lib/minetest/.minetest/worlds
      # Mods
      - ./mods:/var/lib/minetest/.minetest/mods
      # Archivo de configuración
      - ./minetest.conf:/etc/minetest/minetest.conf:ro # :ro = read-only
      # Logs y debug
      - ./debug.txt:/var/lib/minetest/debug.txt

    # Comando para iniciar el servidor
    # Sobrescribe el CMD del Dockerfile para asegurar que usa nuestro config
    command: ["--config", "/etc/minetest/minetest.conf"]

volumes:
  minetest_data:
```

### Puntos Clave de esta Configuración:

*   **`build: .`**: Le dice a Docker Compose que use el `Dockerfile` en el directorio actual para construir la imagen del servidor. Esto es genial porque cualquier cambio que hagas en el `Dockerfile` se aplicará la próxima vez que construyas.
*   **`restart: unless-stopped`**: Si el servidor crashea o el VPS se reinicia, Docker intentará levantarlo automáticamente.
*   **`volumes`**: Esta es la parte más importante. "Mapeamos" carpetas de nuestro VPS (donde clonamos el repo) a carpetas dentro del contenedor.
    *   `./worlds` -> `.../worlds`: Tus mundos se guardan fuera del contenedor. ¡No los perderás si borras el contenedor!
    *   `./mods` -> `.../mods`: Puedes añadir o quitar mods fácilmente desde tu VPS.
    *   `./minetest.conf` -> `.../minetest.conf`: Tu archivo de configuración principal, fácil de editar.

## 4. Flujo de Trabajo y Despliegue

### Configuración Inicial

1.  **Clona tu fork en el VPS:**
    ```bash
    git clone https://github.com/gabrielpantoja-cl/luanti-v.git Luanti
    cd Luanti
    ```
2.  **Crea tu archivo `minetest.conf`:** Puedes copiar el `minetest.conf.example` y ajustarlo. Asegúrate de configurar `server_name`, `server_description`, y `server_announce = true`.
3.  **Descarga un juego:** Luanti necesita un juego base. Descárgalo y ponlo en la carpeta `games`. Por ejemplo, Minetest Game:
    ```bash
    cd games
    git clone https://github.com/minetest/minetest_game.git
    cd ..
    ```
4.  **Construye e inicia el servidor por primera vez:**
    ```bash
    docker-compose build
    docker-compose up -d
    ```
    La primera construcción tardará un rato. Las siguientes serán mucho más rápidas.

### Actualizando el Servidor

Cuando quieras actualizar a la última versión de Luanti (después de sincronizar tu fork con el `upstream`):

1.  **Trae los últimos cambios:**
    ```bash
    git pull origin main # O la rama que uses
    ```
2.  **Reconstruye la imagen y reinicia el servidor:**
    ```bash
    docker-compose build
    docker-compose up -d
    ```
    Docker es lo suficientemente inteligente para solo reconstruir las partes que han cambiado. El `-d` (detached) asegura que el servidor se reinicie en segundo plano.

### Monitoreo y Logs

*   **Ver los logs en tiempo real:**
    ```bash
    docker-compose logs -f
    ```
*   **Ver el uso de recursos:**
    ```bash
    docker stats
    ```

## 5. Backups (¡Crítico!)

Inspirado en el script de `otro-proyecto`, podemos crear un script simple para hacer backups.

**`backup_luanti.sh` (colocar en `server_setup-gabriel`):**
```bash
#!/bin/bash
BACKUP_DIR="/home/gabriel/luanti_backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

mkdir -p $BACKUP_DIR

# Comprimir el mundo y la configuración
tar -czf $BACKUP_DIR/luanti_data_$DATE.tar.gz -C /home/gabriel/Luanti worlds/ minetest.conf mods/

echo "Backup de Luanti creado en $BACKUP_DIR/luanti_data_$DATE.tar.gz"

# Limpiar backups antiguos
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
```

Puedes automatizar este script para que se ejecute diariamente usando `cron`.

## Conclusión

Al combinar la potencia de Docker y Docker Compose con las prácticas de infraestructura como código, hemos creado un sistema para tu servidor Luanti que es:

*   **Robusto:** Se reinicia solo y los datos persisten.
*   **Fácil de mantener:** Actualizar es tan simple como `git pull` y `docker-compose build`.
*   **Seguro:** Aislado en un contenedor y protegido por un firewall.
*   **Escalable:** Listo para crecer cuando tu comunidad lo haga.

Esta configuración te da una base profesional para administrar tu servidor de juegos a largo plazo.
