# 🚀 Despliegue Multi-Aplicación - VPS Escalable

## 📋 Resumen

Esta guía explica cómo configurar tu VPS para manejar múltiples aplicaciones de forma eficiente, usando Docker Compose, GitHub Actions y un sistema de proxy centralizado con Nginx.

## 🏗️ Arquitectura Multi-Aplicación

```
🌍 Internet (80/443)
     ↓
🔒 Nginx Centralizado
     ├─ pitutito.cl → Docker :3000
     ├─ studio.pitutito.cl → Docker :8005  
     ├─ app2.com → Docker :3001
     └─ app3.com → Docker :3002
     ↓
🐳 Docker Shared Network
     ├─ pitutito_network (app específica)
     ├─ app2_network (app específica)
     └─ shared_apps_network (compartida)
     ↓
💾 Recursos Compartidos
     ├─ Logs centralizados
     ├─ SSL/certificados
     └─ Scripts de utilidad
```

## 🎯 Ventajas de esta Arquitectura

### ✅ **Escalabilidad**
- Fácil agregar nuevas aplicaciones
- Recursos compartidos eficientemente
- Configuración modular

### ✅ **Mantenimiento**
- Configuración centralizada de Nginx
- Logs unificados
- Scripts de utilidad compartidos

### ✅ **Seguridad**
- SSL centralizado
- Firewall simplificado
- Redes Docker aisladas

### ✅ **Monitoreo**
- Vista unificada de todas las aplicaciones
- Alertas centralizadas
- Backup automatizado

## 🚀 Configuración Inicial

### 1. **Configurar VPS para Multi-App**

```bash
# Desde tu máquina local
./setup-multi-app-vps.sh
```

Este script configurará:
- 📁 Estructura de directorios para múltiples apps
- 🌐 Red Docker compartida
- ⚙️ Configuración centralizada de Nginx
- 📊 Scripts de monitoreo y utilidad
- 📝 Sistema de logs centralizado

### 2. **Configurar Repositorio de Pitutito**

```bash
# Configurar repositorio Git en VPS
./setup-vps-repository.sh
```

### 3. **Configurar GitHub Secrets**

Ve a tu repositorio → Settings → Secrets and variables → Actions:

- `VPS_HOST`: `167.172.251.27`
- `VPS_USER`: `gabriel`  
- `VPS_SSH_KEY`: Tu clave SSH privada

## 📱 Estructura de Directorios

```
/home/gabriel/apps/
├── pitutito/                 # Aplicación Pitutito
│   ├── .git/
│   ├── docker-compose.production.yml
│   ├── .env.production
│   └── src/
├── shared/                   # Recursos compartidos
│   ├── nginx/
│   │   ├── multi-app.conf    # Configuración base
│   │   ├── upstreams/        # Upstreams por app
│   │   └── sites/            # Sitios virtuales
│   └── scripts/
│       ├── add-app.sh        # Agregar nueva app
│       └── monitor.sh        # Monitoreo
├── logs/                     # Logs centralizados
│   ├── nginx/
│   ├── apps/
│   └── system/
└── backups/                  # Backups automatizados
```

## ➕ Agregar Nueva Aplicación

### Paso 1: Usar Script Automatizado

```bash
# En el VPS
/home/gabriel/apps/shared/scripts/add-app.sh myapp myapp.com 3001
```

### Paso 2: Configurar SSL

```bash
# Obtener certificado SSL
sudo certbot --nginx -d myapp.com
```

### Paso 3: Crear Docker Compose

```yaml
# /home/gabriel/apps/myapp/docker-compose.production.yml
version: '3.8'

networks:
  myapp_network:
    driver: bridge
  shared_network:
    external: true
    name: shared_apps_network

services:
  myapp-web:
    build: .
    container_name: myapp_web
    ports:
      - "3001:80"
    networks:
      - myapp_network
      - shared_network
    restart: unless-stopped
    labels:
      - "app.name=myapp"
      - "app.environment=production"
```

### Paso 4: Configurar GitHub Actions

```yaml
# .github/workflows/deploy.yml para nueva app
# Cambiar solo:
PROJECT_DIR="/home/gabriel/apps/myapp"  # ← Nuevo directorio
# Y el puerto en health checks
```

## 📊 Monitoreo y Mantenimiento

### **Ver Estado General**

```bash
# En el VPS
/home/gabriel/apps/shared/scripts/monitor.sh
```

### **Ver Logs Específicos**

```bash
# Logs de Nginx
tail -f /home/gabriel/apps/logs/nginx/access.log

# Logs de aplicación específica
cd /home/gabriel/apps/pitutito
docker compose -f docker-compose.production.yml logs -f

# Logs de sistema
journalctl -f -u nginx
```

### **Comandos de Mantenimiento**

```bash
# Reiniciar aplicación específica
cd /home/gabriel/apps/pitutito
docker compose -f docker-compose.production.yml restart

# Recargar Nginx (sin downtime)
sudo systemctl reload nginx

# Ver uso de recursos
docker stats

# Limpiar Docker
docker system prune -f
```

## 🔧 Configuraciones Avanzadas

### **Load Balancing** (Para alta demanda)

```nginx
# En upstream configuration
upstream pitutito_backend {
    server 127.0.0.1:3000 weight=3;
    server 127.0.0.1:3100 weight=2;  # Segunda instancia
    keepalive 32;
}
```

### **Rate Limiting Por Aplicación**

```nginx
# Diferentes límites por aplicación
location /api/ {
    limit_req zone=api burst=100 nodelay;
    proxy_pass http://pitutito_backend;
}
```

### **Backup Automatizado**

```bash
# Crear script de backup para todas las apps
# /home/gabriel/apps/shared/scripts/backup-all.sh
```

## 🚨 Troubleshooting

### **Problema: Nueva app no responde**

```bash
# Verificar puerto ocupado
ss -tlnp | grep :3001

# Verificar configuración Nginx
sudo nginx -t

# Ver logs de la app
docker logs <container_name>
```

### **Problema: SSL no funciona**

```bash
# Verificar certificados
sudo certbot certificates

# Renovar manualmente
sudo certbot renew

# Verificar configuración SSL en Nginx
openssl s_client -connect myapp.com:443
```

### **Problema: Docker network**

```bash
# Recrear red compartida
docker network rm shared_apps_network
docker network create shared_apps_network

# Verificar conectividad entre containers
docker exec -it container1 ping container2
```

## 📈 Casos de Uso

### **Ejemplo: E-commerce + Blog + API**

```
📱 ecommerce.com:443 → Docker :3000 (React App)
📝 blog.ecommerce.com:443 → Docker :3001 (WordPress)
🔌 api.ecommerce.com:443 → Docker :3002 (Node.js API)
```

### **Ejemplo: Múltiples Clientes**

```
🏢 cliente1.com:443 → Docker :3000
🏢 cliente2.com:443 → Docker :3001  
🏢 cliente3.com:443 → Docker :3002
```

## 💡 Best Practices

### **Nomenclatura Consistente**
- Ports: 3000, 3001, 3002, etc.
- Networks: `<app>_network`
- Containers: `<app>_web`, `<app>_db`

### **Seguridad**
- ✅ Firewall configurado (solo 22, 80, 443)
- ✅ SSL para todas las aplicaciones
- ✅ Redes Docker aisladas por aplicación
- ✅ Backups regulares

### **Performance**
- ✅ Nginx proxy con keepalive
- ✅ Gzip habilitado
- ✅ Rate limiting configurado
- ✅ Logs rotados automáticamente

## 🎯 Próximos Pasos

Una vez que tengas el sistema básico funcionando:

1. **🔄 CI/CD Avanzado**: Pipeline para múltiples apps
2. **📊 Monitoring**: Prometheus + Grafana
3. **🔄 Load Balancing**: Múltiples instancias por app
4. **☁️ CDN**: CloudFlare para assets estáticos
5. **📦 Registry**: Docker Registry privado

---

**✅ Con esta configuración podrás escalar de 1 aplicación a 10+ aplicaciones en el mismo VPS de forma eficiente** 🚀