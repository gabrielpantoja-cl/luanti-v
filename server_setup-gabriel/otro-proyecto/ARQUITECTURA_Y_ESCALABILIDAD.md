# 🏗️ Infraestructura Digital Ocean - Pitutito.cl

**Documentación técnica de la infraestructura para la plataforma Pituto Vecino Confiable**

*Desarrollado por Gabriel Pantoja*

---

## 🎯 Resumen Ejecutivo

Esta documentación describe la arquitectura completa para alojar **pitutito.cl** y 3 proyectos adicionales en Digital Ocean, con un costo inicial de **$21/mes** y capacidad de escalar según necesidades.

## 🏛️ Arquitectura General

### Proveedor de Cloud

Se ha seleccionado DigitalOcean como el proveedor de servicios en la nube para alojar los proyectos.

### Servidor / Droplet

**Estrategia de Deployment Escalonado**

**Fase 1: Plan Inicial (Costo-Eficiente)**

Para empezar de manera económica y escalar según necesidad:

-   **Tipo:** Droplet Básico Premium AMD
-   **CPU:** 2 vCPUs
-   **Memoria RAM:** 2 GiB
-   **Almacenamiento NVMe:** 60 GB
-   **Transferencia:** 3 TB
-   **Costo:** $21/mes

**Servicios iniciales en esta fase:**
- 1-2 sitios web principales (pitutito.cl)
- Supabase auto-alojado (configuración mínima)
- Monitoreo de recursos (htop, netdata)

**Fase 2: Plan de Crecimiento (Cuando sea necesario)**

-   **Tipo:** Droplet Básico Premium AMD
-   **CPU:** 2 vCPUs
-   **Memoria RAM:** 8 GiB
-   **Almacenamiento NVMe:** 100 GB
-   **Transferencia:** 5 TB
-   **Costo:** $42/mes

**Cuándo escalar a Fase 2:**
- Uso de RAM consistentemente >80%
- Al agregar n8n al stack
- Al desplegar los 4 sitios web completos
- Antes de implementar funcionalidades de RAG

**Ventajas de esta estrategia:**
- ✅ Costo inicial 50% menor que alternativas (Supabase + Vercel = $45/mes)
- ✅ Escalabilidad sin migración de datos
- ✅ Aprender y experimentar sin compromiso económico alto
- ✅ Posibilidad de downgrade si no se necesita la capacidad extra

### Futura Escalabilidad (Fase RAG)

La implementación de la funcionalidad de **Retrieval-Augmented Generation (RAG)** es intensiva en recursos de CPU y, sobre todo, de RAM.

-   **Opción 1 (Escalado Vertical):** Escalar el Droplet actual a una instancia con **al menos 16 GiB de RAM**.
-   **Opción 2 (Servidor Dedicado para IA):** Considerar un Droplet optimizado para CPU o, preferiblemente, una **instancia de GPU** para un rendimiento óptimo del modelo de lenguaje.

---

## 🔧 Stack Tecnológico

### Contenedorización
- **Docker**: Containerización de aplicaciones
- **Docker Compose**: Orquestación de servicios
- **Nginx**: Reverse proxy y servidor web

### Base de Datos
- **PostgreSQL 15**: Base de datos principal
- **Redis 7**: Cache y sesiones
- **Backups**: Automatizados con pg_dump

### Monitoreo y Seguridad
- **Let's Encrypt**: Certificados SSL gratuitos
- **UFW**: Firewall Ubuntu
- **Fail2ban**: Protección contra ataques
- **Logrotate**: Gestión de logs

---

## 📁 Estructura del Servidor

```
/var/www/
├── pitutito.cl/
│   ├── docker-compose.yml
│   ├── nginx.conf
│   ├── frontend/
│   │   ├── dist/          # Build de React
│   │   └── src/
│   ├── backend/
│   │   ├── src/
│   │   ├── Dockerfile
│   │   └── package.json
│   └── database/
│       ├── migrations/
│       └── seeds/
├── proyecto2/
├── proyecto3/
└── proyecto4/
```

---

## 🐳 Configuración Docker

### docker-compose.yml
```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: pitutito_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: pitutito_db
      POSTGRES_USER: pitutito_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=es_ES.UTF-8"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/migrations:/docker-entrypoint-initdb.d/
    ports:
      - "127.0.0.1:5432:5432"
    networks:
      - pitutito_network

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: pitutito_redis
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "127.0.0.1:6379:6379"
    networks:
      - pitutito_network

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: pitutito_backend
    restart: unless-stopped
    ports:
      - "127.0.0.1:3001:3001"
    environment:
      NODE_ENV: production
      PORT: 3001
      DATABASE_URL: postgresql://pitutito_user:${DB_PASSWORD}@postgres:5432/pitutito_db
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      JWT_SECRET: ${JWT_SECRET}
      SPACES_ENDPOINT: ${SPACES_ENDPOINT}
      SPACES_KEY: ${SPACES_KEY}
      SPACES_SECRET: ${SPACES_SECRET}
    depends_on:
      - postgres
      - redis
    networks:
      - pitutito_network
    volumes:
      - ./uploads:/app/uploads

volumes:
  postgres_data:
  redis_data:

networks:
  pitutito_network:
    driver: bridge
```

### Backend Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Instalar dependencias
COPY package*.json ./
RUN npm ci --only=production

# Copiar código fuente
COPY . .

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3001

CMD ["npm", "start"]
```

---

## 🌐 Configuración Nginx

### nginx.conf
```nginx
# Configuración principal para pitutito.cl
upstream pitutito_backend {
    server 127.0.0.1:3001;
}

# Redirección HTTP a HTTPS
server {
    listen 80;
    server_name pitutito.cl www.pitutito.cl;
    return 301 https://pitutito.cl$request_uri;
}

# Servidor HTTPS principal
server {
    listen 443 ssl http2;
    server_name pitutito.cl www.pitutito.cl;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/pitutito.cl/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/pitutito.cl/privkey.pem;
    
    # SSL Security
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # Security Headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Frontend estático
    location / {
        root /var/www/pitutito.cl/frontend/dist;
        try_files $uri $uri/ /index.html;
        
        # Cache para assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # API Backend
    location /api/ {
        proxy_pass http://pitutito_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # WebSocket para chat en tiempo real
    location /ws {
        proxy_pass http://pitutito_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }

    # Uploads
    location /uploads/ {
        alias /var/www/pitutito.cl/uploads/;
        expires 1y;
        add_header Cache-Control "public";
    }
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;
```

---

## 🗄️ Esquema de Base de Datos

### Tablas Principales
```sql
-- Usuarios
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    avatar_url VARCHAR(255),
    phone VARCHAR(20),
    location VARCHAR(100),
    verified BOOLEAN DEFAULT false,
    rating DECIMAL(2,1) DEFAULT 0.0,
    total_reviews INTEGER DEFAULT 0,
    bio TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Pitutos (Servicios)
CREATE TABLE pitutos (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(50) NOT NULL,
    budget VARCHAR(50) NOT NULL,
    location VARCHAR(100) NOT NULL,
    user_id INTEGER NOT NULL REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'completed', 'cancelled', 'in_progress')),
    views INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Reviews y Ratings
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    pituto_id INTEGER NOT NULL REFERENCES pitutos(id),
    reviewer_id INTEGER NOT NULL REFERENCES users(id),
    reviewed_id INTEGER NOT NULL REFERENCES users(id),
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Sistema de Mensajes
CREATE TABLE conversations (
    id SERIAL PRIMARY KEY,
    pituto_id INTEGER NOT NULL REFERENCES pitutos(id),
    requester_id INTEGER NOT NULL REFERENCES users(id),
    provider_id INTEGER NOT NULL REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    conversation_id INTEGER NOT NULL REFERENCES conversations(id),
    sender_id INTEGER NOT NULL REFERENCES users(id),
    content TEXT NOT NULL,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categorías
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    icon VARCHAR(50),
    color VARCHAR(7) -- HEX color
);

-- Índices para performance
CREATE INDEX idx_pitutos_category ON pitutos(category);
CREATE INDEX idx_pitutos_location ON pitutos(location);
CREATE INDEX idx_pitutos_status ON pitutos(status);
CREATE INDEX idx_pitutos_created_at ON pitutos(created_at);
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_reviews_reviewed_id ON reviews(reviewed_id);
```

---

## 🚀 API Endpoints

### Autenticación
```
POST   /api/auth/register     # Registro de usuario
POST   /api/auth/login        # Login
POST   /api/auth/logout       # Logout
GET    /api/auth/me           # Perfil actual
PUT    /api/auth/me           # Actualizar perfil
POST   /api/auth/refresh      # Refresh token
```

### Pitutos (Servicios)
```
GET    /api/pitutos           # Listar pitutos (con filtros)
POST   /api/pitutos           # Crear pituto
GET    /api/pitutos/:id       # Obtener pituto específico
PUT    /api/pitutos/:id       # Actualizar pituto
DELETE /api/pitutos/:id       # Eliminar pituto
POST   /api/pitutos/:id/apply # Aplicar a pituto
```

### Usuarios
```
GET    /api/users/:id         # Perfil público
GET    /api/users/:id/reviews # Reviews del usuario
POST   /api/users/:id/review  # Crear review
```

### Mensajes
```
GET    /api/conversations     # Conversaciones del usuario
POST   /api/conversations     # Iniciar conversación
GET    /api/conversations/:id/messages # Mensajes
POST   /api/conversations/:id/messages # Enviar mensaje
```

### Upload de Archivos
```
POST   /api/upload/avatar     # Subir avatar
POST   /api/upload/pituto     # Subir imagen de pituto
```

---

## 📊 Monitoreo y Logs

### Plan de Monitoreo y Escalabilidad

**Métricas Clave a Monitorear**

**Recursos del Sistema:**
- **RAM**: Mantener uso <80% para operación estable
- **CPU**: Picos sostenidos >70% indican necesidad de más vCPUs
- **Disco**: >85% de uso requiere limpieza o escalado de almacenamiento
- **Red**: Monitorear transferencia mensual vs límite de 3TB

**Herramientas de Monitoreo Recomendadas:**
- `htop`: Monitoreo en tiempo real de CPU y RAM
- `netdata`: Dashboard web con métricas históricas
- `df -h`: Verificación rápida de espacio en disco
- `vnstat`: Monitoreo de uso de ancho de banda

**Criterios de Escalabilidad**

**Escalar de Fase 1 ($21) a Fase 2 ($42) cuando:**
1. RAM consistentemente >80% por más de una semana
2. Páginas web presentan lentitud por falta de recursos
3. Supabase muestra errores de conexión por memoria
4. Se quiere agregar n8n o más servicios

**Escalar a Plan Superior (Fase RAG) cuando:**
1. Se implementa funcionalidad de RAG
2. Se requiere >8 GiB de RAM consistentemente
3. Se necesita procesamiento intensivo de IA/ML

**Proceso de Escalado**

1. **Crear snapshot de seguridad** antes del escalado
2. **Programar escalado** en horarios de bajo tráfico
3. **Validar servicios** después del escalado
4. **Monitorear rendimiento** post-escalado

### Scripts de Monitoreo
```bash
#!/bin/bash
# /home/gabriel/scripts/monitor-pitutito.sh

# Verificar servicios críticos
services=("postgres" "redis" "nginx" "pitutito_backend")

for service in "${services[@]}"; do
    if ! docker ps | grep -q "$service"; then
        echo "⚠️  $service is down!" | mail -s "Pitutito Alert" gabriel@email.com
        docker-compose -f /var/www/pitutito.cl/docker-compose.yml restart "$service"
    fi
done

# Verificar espacio en disco
disk_usage=$(df /var/www | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$disk_usage" -gt 85 ]; then
    echo "⚠️ Disk usage is at $disk_usage%" | mail -s "Disk Space Alert" gabriel@email.com
fi

# Verificar memoria
memory_usage=$(free | grep Mem | awk '{printf("%.2f", $3/$2 * 100.0)}')
if (( $(echo "$memory_usage > 90.0" | bc -l) )); then
    echo "⚠️ Memory usage is at $memory_usage%" | mail -s "Memory Alert" gabriel@email.com
fi
```

### Configuración de Logs
```bash
# /etc/logrotate.d/pitutito
/var/www/pitutito.cl/logs/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0644 www-data www-data
    postrotate
        docker-compose -f /var/www/pitutito.cl/docker-compose.yml restart backend
    endscript
}
```

---

## 🔐 Seguridad

### Firewall (UFW)
```bash
# Configuración básica de firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### Fail2ban
```ini
# /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log

[pitutito-api]
enabled = true
filter = pitutito-api
logpath = /var/www/pitutito.cl/logs/backend.log
maxretry = 10
bantime = 1800
```

---

## 💰 Costos y Escalabilidad

### Costos Actuales
| Servicio | Costo Mensual | Descripción |
|----------|---------------|-------------|
| Droplet 2GB | $21.00 | Servidor principal |
| **Total** | **$21.00** | **Fase 1** |

### Plan de Escalamiento

#### Fase 1: MVP (0-1k usuarios)
- **Actual**: 2GB RAM, $21/mes
- **Capacidad**: ~100-200 usuarios concurrentes

#### Fase 2: Crecimiento (1k-10k usuarios)
- **Upgrade**: 8GB RAM, $42/mes
- **Capacidad**: ~500-1000 usuarios concurrentes

#### Fase 3: Escala (10k+ usuarios)
- **Múltiples droplets**: 2x 8GB RAM
- **Base de datos dedicada**: Managed Database $75/mes
- **CDN Premium**: $20/mes
- **Total**: ~$180/mes

---

## 🛠️ Scripts de Deployment

### Deployment Automático
```bash
#!/bin/bash
# deploy.sh - Script de deployment para pitutito.cl

set -e

PROJECT_DIR="/var/www/pitutito.cl"
BACKUP_DIR="/var/backups/pitutito"
DATE=$(date +%Y%m%d_%H%M%S)

echo "🚀 Starting deployment..."

# Crear backup
echo "📦 Creating backup..."
mkdir -p $BACKUP_DIR
pg_dump pitutito_db > $BACKUP_DIR/db_backup_$DATE.sql

# Pull latest changes
cd $PROJECT_DIR
git pull origin main

# Build frontend
echo "🏗️ Building frontend..."
cd frontend
npm ci
npm run build

# Update backend
echo "🔧 Updating backend..."
cd ../backend
npm ci

# Restart services
echo "🔄 Restarting services..."
docker-compose down
docker-compose up -d --build

# Health check
echo "🏥 Running health check..."
sleep 30
if curl -f http://localhost:3001/api/health > /dev/null 2>&1; then
    echo "✅ Deployment successful!"
else
    echo "❌ Health check failed, rolling back..."
    docker-compose down
    # Restore from backup if needed
    exit 1
fi

echo "🎉 Deployment completed successfully!"
```

### Backup Script
```bash
#!/bin/bash
# backup.sh - Backup completo de pitutito.cl

BACKUP_DIR="/var/backups/pitutito"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p $BACKUP_DIR

# Database backup
pg_dump -h localhost -p 5432 -U pitutito_user pitutito_db | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# Files backup
tar -czf $BACKUP_DIR/files_$DATE.tar.gz -C /var/www/pitutito.cl uploads/

# Upload to Spaces (opcional)
aws s3 cp $BACKUP_DIR/db_$DATE.sql.gz s3://pitutito-backups/database/ --endpoint-url=https://nyc3.digitaloceanspaces.com
aws s3 cp $BACKUP_DIR/files_$DATE.tar.gz s3://pitutito-backups/files/ --endpoint-url=https://nyc3.digitaloceanspaces.com

# Cleanup old backups
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete

echo "✅ Backup completed: $DATE"
```

---

## 📋 Checklist de Deployment

### Pre-deployment
- [ ] Configurar DNS para pitutito.cl
- [ ] Crear cuenta Digital Ocean
- [ ] Configurar SSH keys
- [ ] Registrar dominio y configurar DNS

### Initial Setup
- [ ] Crear y configurar Droplet
- [ ] Instalar Docker y Docker Compose
- [ ] Configurar Nginx
- [ ] Obtener certificados SSL
- [ ] Configurar firewall y seguridad

### Application Setup
- [ ] Clonar repositorio
- [ ] Configurar variables de entorno
- [ ] Levantar servicios con Docker Compose
- [ ] Ejecutar migraciones de BD
- [ ] Cargar datos semilla

### Testing & Monitoring
- [ ] Verificar endpoints API
- [ ] Probar frontend en producción
- [ ] Configurar monitoreo
- [ ] Setup backups automáticos
- [ ] Configurar alertas

### Go Live
- [ ] DNS switch a producción
- [ ] Monitoring activo
- [ ] Performance testing
- [ ] User acceptance testing

---

## 📞 Contacto y Soporte

**Desarrollador**: Gabriel Pantoja  
**Email**: gabriel@pitutito.cl  
**Infraestructura**: Digital Ocean  
**Monitoreo**: 24/7 automatizado

---

*Documentación actualizada: Agosto 2025*  
*Versión: 2.0*
