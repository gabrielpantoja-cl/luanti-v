# Verificación de la Instalación de Supabase Self-Hosted

Este documento contiene los comandos esenciales para verificar que tu instancia autoalojada de Supabase esté funcionando correctamente en el servidor.

**IMPORTANTE:** Todos los comandos deben ejecutarse desde el directorio de configuración de Docker de Supabase, que generalmente es `~/supabase/docker`.

---

### 1. Navegar al Directorio Correcto

Antes de ejecutar cualquier comando de verificación, asegúrate de estar en el directorio correcto.

```bash
cd ~/supabase/docker
```

---

### 2. Revisar el Estado de los Contenedores

El comando más importante. Muestra todos los servicios (contenedores) que gestiona Supabase y su estado actual.

```bash
docker compose ps
```

**Qué esperar:**
Deberías ver una lista de aproximadamente 15-20 servicios. La columna `Status` debería mostrar `running` o `healthy` para todos ellos. Si alguno muestra `exited` o `restarting`, es una señal de que algo anda mal.

**Ejemplo de salida esperada (parcial):**
```
NAME                          STATUS                   PORTS
supabase-auth                 running(healthy)         0.0.0.0:9999->9999/tcp
supabase-db                   running(healthy)         0.0.0.0:5432->5432/tcp
supabase-kong                 running                  0.0.0.0:8000->8000/tcp, 0.0.0.0:8443->8443/tcp
supabase-studio               running                  
...
```

---

### 3. Revisar los Logs (Registros)

Si un contenedor no está funcionando correctamente, los logs son el mejor lugar para encontrar errores.

**Para ver los logs de todos los servicios (últimas 100 líneas):**
```bash
docker compose logs --tail=100
```

**Para seguir los logs en tiempo real:**
```bash
docker compose logs -f
```

**Para revisar los logs de un servicio específico (ej. la base de datos):**
Si sospechas que un servicio en particular está fallando (por ejemplo, `db`), puedes ver sus logs de forma aislada.
```bash
docker compose logs db
```

---

### 4. Acceder a Supabase Studio (El Panel de Control)

La prueba definitiva es acceder a la interfaz gráfica de Supabase.

1.  **URL de Acceso:** `http://<IP_DE_TU_SERVIDOR>:8000`
2.  **Credenciales por defecto:**
    *   **email:** `supabase@example.com`
    *   **password:** `this-is-a-safe-password`

    *(Estas credenciales se pueden cambiar en el archivo de configuración `docker-compose.yml` si lo deseas)*.

Si puedes iniciar sesión y ver el panel de control de Supabase, ¡tu instancia está instalada y funcionando correctamente!

---

### Acceso Directo a la Base de Datos (pitutito.cl)

Para acceder directamente a la sección de tablas de la base de datos para el proyecto **pitutito.cl**, puedes usar la siguiente URL:

- **URL Directa:** `http://167.172.251.27:8000/project/default/database/tables`

Esto te llevará directamente al editor de tablas dentro de Supabase Studio.

---

### 5. Conexión Directa a la Base de Datos (Opcional, para depuración)
