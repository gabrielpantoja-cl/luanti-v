# Configuración Docker Multi-Proyecto

## Estructura de Directorios Recomendada

```
~/multi-projects/
├── docker-compose.yml                 # Orquestador principal
├── .env                              # Variables globales
├── nginx/
│   └── nginx.conf                    # Proxy reverso
├── gabrielpantoja-blog/
│   ├── .env                         # Variables específicas
│   └── docker-compose.supabase.yml  # Supabase dedicado
├── referenciales-db/
│   ├── .env
│   └── docker-compose.supabase.yml
├── pantoja-propiedades/
│   ├── .env  
│   └── docker-compose.supabase.yml
└── pitutito-trabajos/
    ├── .env
    └── docker-compose.supabase.yml
```

## Docker Compose Principal

```yaml
# ~/multi-projects/docker-compose.yml
version: '3.8'

networks:
  multi_project_network:
    driver: bridge

services:
  # Nginx Proxy Reverso
  nginx:
    image: nginx:alpine
    container_name: nginx_proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    networks:
      - multi_project_network
    restart: unless-stopped
    depends_on:
      - gabrielpantoja-supabase
      - referenciales-supabase
      - pantoja-propiedades-supabase
      - pitutito-supabase

  # PROYECTO 1: GABRIELPANTOJA.CL (Blog Personal)
  gabrielpantoja-supabase:
    extends:
      file: ./gabrielpantoja-blog/docker-compose.supabase.yml
      service: supabase-all-in-one
    container_name: gabrielpantoja_supabase
    ports:
      - "8001:8000"  # Dashboard
      - "54321:5432" # PostgreSQL
    environment:
      - PROJECT_NAME=gabrielpantoja
      - POSTGRES_DB=gabrielpantoja_blog_db
    networks:
      - multi_project_network
    volumes:
      - gabrielpantoja_db_data:/var/lib/postgresql/data
      - gabrielpantoja_storage_data:/var/lib/storage

  # PROYECTO 2: REFERENCIALES.CL (Base de Datos Colaborativa)
  referenciales-supabase:
    extends:
      file: ./referenciales-db/docker-compose.supabase.yml
      service: supabase-all-in-one
    container_name: referenciales_supabase
    ports:
      - "8002:8000"  # Dashboard
      - "54322:5432" # PostgreSQL
    environment:
      - PROJECT_NAME=referenciales
      - POSTGRES_DB=referenciales_db
    networks:
      - multi_project_network
    volumes:
      - referenciales_db_data:/var/lib/postgresql/data
      - referenciales_storage_data:/var/lib/storage

  # PROYECTO 3: PANTOJAPROPIEDADES.CL (Corredora de Propiedades)
  pantoja-propiedades-supabase:
    extends:
      file: ./pantoja-propiedades/docker-compose.supabase.yml
      service: supabase-all-in-one
    container_name: pantoja_propiedades_supabase
    ports:
      - "8003:8000"  # Dashboard
      - "54323:5432" # PostgreSQL
    environment:
      - PROJECT_NAME=pantoja_propiedades
      - POSTGRES_DB=pantoja_propiedades_db
    networks:
      - multi_project_network
    volumes:
      - pantoja_propiedades_db_data:/var/lib/postgresql/data
      - pantoja_propiedades_storage_data:/var/lib/storage

  # PROYECTO 4: PITUTITO.CL (Trabajos Informales por Ubicación)
  pitutito-supabase:
    extends:
      file: ./pitutito-trabajos/docker-compose.supabase.yml
      service: supabase-all-in-one
    container_name: pitutito_supabase
    ports:
      - "8004:8000"  # Dashboard
      - "54324:5432" # PostgreSQL
    environment:
      - PROJECT_NAME=pitutito
      - POSTGRES_DB=pitutito_trabajos_db
    networks:
      - multi_project_network
    volumes:
      - pitutito_db_data:/var/lib/postgresql/data
      - pitutito_storage_data:/var/lib/storage

volumes:
  # Volúmenes por proyecto (datos aislados)
  gabrielpantoja_db_data:
  gabrielpantoja_storage_data:
  referenciales_db_data:
  referenciales_storage_data:
  pantoja_propiedades_db_data:
  pantoja_propiedades_storage_data:
  pitutito_db_data:
  pitutito_storage_data:
```

## Template Supabase por Proyecto

```yaml
# proyecto1-pitutito/docker-compose.supabase.yml
version: '3.8'

services:
  supabase-all-in-one:
    image: supabase/supabase:latest
    environment:
      # Configuración específica del proyecto
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      ANON_KEY: ${ANON_KEY}
      SERVICE_ROLE_KEY: ${SERVICE_ROLE_KEY}
      DASHBOARD_USERNAME: ${DASHBOARD_USERNAME}
      DASHBOARD_PASSWORD: ${DASHBOARD_PASSWORD}
      # Personalización por proyecto
      STUDIO_DEFAULT_ORGANIZATION: ${PROJECT_NAME}
      STUDIO_DEFAULT_PROJECT: ${PROJECT_NAME}
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Nginx Configuration

```nginx
# nginx/nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream gabrielpantoja_backend {
        server gabrielpantoja-supabase:8000;
    }
    
    upstream referenciales_backend {
        server referenciales-supabase:8000;
    }
    
    upstream pantoja_propiedades_backend {
        server pantoja-propiedades-supabase:8000;
    }
    
    upstream pitutito_backend {
        server pitutito-supabase:8000;
    }

    # PROYECTO 1: gabrielpantoja.cl (Blog Personal de Tasaciones y Tecnología)
    server {
        listen 80;
        server_name gabrielpantoja.cl www.gabrielpantoja.cl;
        
        location / {
            proxy_pass http://gabrielpantoja_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    # PROYECTO 2: referenciales.cl (Base de Datos Colaborativa)
    server {
        listen 80;
        server_name referenciales.cl www.referenciales.cl;
        
        location / {
            proxy_pass http://referenciales_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    # PROYECTO 3: pantojapropiedades.cl (Corredora de Propiedades)
    server {
        listen 80;
        server_name pantojapropiedades.cl www.pantojapropiedades.cl;
        
        location / {
            proxy_pass http://pantoja_propiedades_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    # PROYECTO 4: pitutito.cl (Trabajos Informales por Ubicación)
    server {
        listen 80;
        server_name pitutito.cl www.pitutito.cl;
        
        location / {
            proxy_pass http://pitutito_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    # Acceso directo a dashboards por IP
    server {
        listen 80;
        server_name 167.172.251.27;
        
        location /gabrielpantoja/ {
            proxy_pass http://gabrielpantoja_backend/;
        }
        
        location /referenciales/ {
            proxy_pass http://referenciales_backend/;
        }
        
        location /pantoja-propiedades/ {
            proxy_pass http://pantoja_propiedades_backend/;
        }
        
        location /pitutito/ {
            proxy_pass http://pitutito_backend/;
        }
    }
}
```

## Variables de Entorno por Proyecto

```bash
# gabrielpantoja-blog/.env
PROJECT_NAME=gabrielpantoja
POSTGRES_PASSWORD=gabrielpantoja_super_secret_2025
JWT_SECRET=AiPyxCeTSXAVjDesp0F3xlxFI6db2bfG_gabrielpantoja
ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...gabrielpantoja
SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...gabrielpantoja
DASHBOARD_USERNAME=admin_gabrielpantoja  
DASHBOARD_PASSWORD=secure_password_gabrielpantoja_2025

# referenciales-db/.env
PROJECT_NAME=referenciales
POSTGRES_PASSWORD=referenciales_super_secret_2025
JWT_SECRET=AiPyxCeTSXAVjDesp0F3xlxFI6db2bfG_referenciales
ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...referenciales
SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...referenciales
DASHBOARD_USERNAME=admin_referenciales
DASHBOARD_PASSWORD=secure_password_referenciales_2025

# pantoja-propiedades/.env
PROJECT_NAME=pantoja_propiedades
POSTGRES_PASSWORD=pantoja_propiedades_super_secret_2025
JWT_SECRET=AiPyxCeTSXAVjDesp0F3xlxFI6db2bfG_pantoja_propiedades
ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...pantoja_propiedades
SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...pantoja_propiedades
DASHBOARD_USERNAME=admin_pantoja_propiedades
DASHBOARD_PASSWORD=secure_password_pantoja_propiedades_2025

# pitutito-trabajos/.env
PROJECT_NAME=pitutito
POSTGRES_PASSWORD=pitutito_super_secret_2025
JWT_SECRET=AiPyxCeTSXAVjDesp0F3xlxFI6db2bfG_pitutito
ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...pitutito
SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...pitutito
DASHBOARD_USERNAME=admin_pitutito  
DASHBOARD_PASSWORD=secure_password_pitutito_2025
```

## Comandos de Gestión

```bash
# Iniciar todos los proyectos
docker-compose up -d

# Iniciar solo un proyecto específico
docker-compose up -d gabrielpantoja-supabase      # Blog personal
docker-compose up -d referenciales-supabase       # Base de datos colaborativa
docker-compose up -d pantoja-propiedades-supabase # Corredora de propiedades
docker-compose up -d pitutito-supabase            # Trabajos informales

# Ver logs de un proyecto específico
docker-compose logs -f gabrielpantoja-supabase

# Reiniciar un proyecto
docker-compose restart referenciales-supabase

# Backup de base de datos específica
docker-compose exec gabrielpantoja-supabase pg_dump -U postgres gabrielpantoja_blog_db > gabrielpantoja_backup.sql
docker-compose exec referenciales-supabase pg_dump -U postgres referenciales_db > referenciales_backup.sql
docker-compose exec pantoja-propiedades-supabase pg_dump -U postgres pantoja_propiedades_db > pantoja_propiedades_backup.sql
docker-compose exec pitutito-supabase pg_dump -U postgres pitutito_trabajos_db > pitutito_backup.sql

# Parar todo
docker-compose down

# Parar y eliminar volúmenes (CUIDADO: borra datos)
docker-compose down -v
```

## Acceso a los Dashboards

- **gabrielpantoja.cl** (Blog Personal): http://167.172.251.27:8001
- **referenciales.cl** (Base de Datos Colaborativa): http://167.172.251.27:8002  
- **pantojapropiedades.cl** (Corredora de Propiedades): http://167.172.251.27:8003
- **pitutito.cl** (Trabajos Informales): http://167.172.251.27:8004

## Ventajas de esta Configuración

✅ **Aislamiento completo**: Cada proyecto tiene su propia BD
✅ **Fácil mantenimiento**: Un comando maneja todo
✅ **Backups individuales**: Por proyecto
✅ **Escalabilidad**: Agregar proyectos es fácil
✅ **Desarrollo local**: Misma estructura en tu máquina
✅ **Updates selectivos**: Solo actualiza lo que necesites

## Migración desde tu Setup Actual

1. **Backup** de datos actuales
2. **Crear** nueva estructura de directorios
3. **Migrar** configuración existente
4. **Probar** en paralelo
5. **Switchear** dominios

---

*Creado: Agosto 2025*