# 🚀 Pitutito.cl - Guía Completa de Despliegue en Producción

## 📋 **Estado Final Exitoso - Agosto 2025**

✅ **Aplicación desplegada y funcional:**
- **🌐 Sitio principal:** https://pitutito.cl 
- **📊 Supabase Studio:** https://studio.pitutito.cl
- **🔒 SSL/HTTPS:** Certificados Let's Encrypt activos
- **🔄 Renovación automática:** Configurada

## Resumen del Proyecto

**Pitutito.cl** es una plataforma de trabajos informales (gig economy) que conecta personas que necesitan servicios con proveedores locales. La aplicación está construida con:

- **Frontend**: React + TypeScript + Vite + Tailwind CSS + shadcn/ui
- **Backend**: Supabase autoalojado (PostgreSQL + Auth + Storage + Realtime)
- **Containerización**: Docker Compose
- **Proxy**: Nginx con SSL/HTTPS
- **DNS**: Cloudflare
- **VPS**: DigitalOcean (167.172.251.27)

## 🔧 **Arquitectura Final de Producción**

```
┌─────────────────────────────────────────┐
│              VPS (167.172.251.27)       │
│             Ubuntu 24.04.1 LTS          │
├─────────────────────────────────────────┤
│  🌍 Internet (Port 80/443)             │
│       ↓                                 │
│  🔒 Nginx + SSL (Let's Encrypt)        │
│       ├─ pitutito.cl → 3000            │
│       └─ studio.pitutito.cl → 8005     │
│       ↓                                 │
│  🐳 Docker Containers                   │
│  ├─ 🌐 React App (Port 3000)           │
│  ├─ 📊 Supabase Studio (Port 8005)     │
│  ├─ 🔌 Kong API Gateway (Port 8001)    │
│  └─ 🗄️  PostgreSQL (Port 54324)        │
└─────────────────────────────────────────┘
```

### **Puertos Finales Configurados**
- **80/443:** Nginx (HTTP/HTTPS público)
- **3000:** Aplicación React (Docker interno)  
- **8005:** Supabase Studio (Docker interno)
- **8001:** Kong API Gateway (Docker interno)
- **54324:** PostgreSQL (Docker interno)

## 🛠️ Archivos de Configuración

### Docker Compose
- `docker-compose.production.yml` - Configuración completa de servicios
- `.env.production` - Variables de entorno de producción

### Supabase
- `supabase/kong.yml` - Configuración del API Gateway
- `database/init/` - Scripts de inicialización de base de datos

### Aplicación
- `Dockerfile` - Build multi-stage con Nginx
- `nginx.conf` - Configuración del servidor web

## 🚀 **Proceso Completo de Despliegue Ejecutado**

### **🔍 1. Diagnóstico Inicial (Problemas Encontrados)**

**Problemas identificados:**
- ❌ VPS no respondía en puerto 80/443
- ❌ DNS configurado pero servicios detenidos
- ❌ Nginx sin configuración personalizada

**Comandos de diagnóstico ejecutados:**
```bash
# Verificar DNS
nslookup pitutito.cl                    # ✅ Apuntaba a 167.172.251.27

# Probar conectividad
curl -I http://167.172.251.27          # ❌ Connection timeout
curl -I http://pitutito.cl             # ❌ Connection timeout

# Verificar estado de servicios
sudo systemctl status nginx            # ✅ Running
sudo ss -tlnp | grep -E ':(80|443)'   # ❌ No escuchaba puerto 80

# Localizar aplicación
find /home/gabriel -name "package.json" # ✅ Encontrada en /home/gabriel/apps/pitutito/
```

### **🔧 2. Configuración de Nginx (Solucionado)**

**Problema:** Nginx usaba configuración por defecto sin proxy

**Solución ejecutada:**
```bash
# Crear configuración de sitio
sudo nano /etc/nginx/sites-available/pitutito.cl
```

**Configuración aplicada:**
```nginx
server {
    listen 80;
    server_name pitutito.cl www.pitutito.cl;
    
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name studio.pitutito.cl;
    
    location / {
        proxy_pass http://127.0.0.1:8005;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_Set_header X-Forwarded-Proto $scheme;
    }
}
```

**Activación exitosa:**
```bash
# Habilitar sitio (método que funcionó)
cd /etc/nginx/sites-enabled/
sudo ln -sf ../sites-available/pitutito.cl .

# Verificar configuración y recargar
sudo nginx -t && sudo systemctl reload nginx

# Verificación exitosa
sudo ss -tlnp | grep :80  # ✅ Puerto 80 activo
curl -H "Host: pitutito.cl" localhost:80  # ✅ HTML de React recibido
```

### **🔒 3. Configuración SSL/HTTPS (Solucionado)**

**Problema:** Firewall bloqueaba Let's Encrypt

**Diagnóstico:**
```bash
sudo ufw status  # Solo SSH permitido
```

**Solución de firewall:**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp  
sudo ufw allow 'Nginx Full'
```

**Instalación de Certbot:**
```bash
sudo apt update
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/bin/certbot
```

**Obtención exitosa de certificados:**
```bash
sudo certbot --nginx \
    -d pitutito.cl \
    -d www.pitutito.cl \
    -d studio.pitutito.cl \
    --agree-tos \
    --no-eff-email \
    --email admin@pitutito.cl \
    --non-interactive
```

**Renovación automática configurada:**
```bash
sudo systemctl enable snap.certbot.renew.timer
sudo systemctl start snap.certbot.renew.timer
```

## 🌐 **URLs Finales de Producción**

- **🏠 Sitio principal:** https://pitutito.cl
- **🌍 Con www:** https://www.pitutito.cl  
- **📊 Supabase Studio:** https://studio.pitutito.cl

### **URLs de Desarrollo/API (Internas)**
- **API Gateway**: http://167.172.251.27:8001
- **PostgreSQL**: 167.172.251.27:54324

## ✅ **Verificación Final Exitosa**

### **Pruebas realizadas:**
```bash
# Desde el VPS
curl -H "Host: pitutito.cl" localhost:80        # ✅ HTML de React
curl -H "Host: studio.pitutito.cl" localhost:80 # ✅ Redirect de Supabase

# Desde internet  
https://pitutito.cl         # ✅ Aplicación cargando
https://studio.pitutito.cl  # ✅ Supabase Studio funcional
```

## 🔒 **Configuración de Seguridad Final**

### **Firewall (UFW):**
```bash
sudo ufw status
```
**Reglas aplicadas:**
- ✅ SSH (22)
- ✅ HTTP (80)  
- ✅ HTTPS (443)
- ✅ Nginx Full

### **SSL/TLS:**
- ✅ Certificados Let's Encrypt válidos para 3 dominios
- ✅ Renovación automática configurada (90 días)
- ✅ HTTPS forzado por Certbot

## 🗄️ Base de Datos

La aplicación crea automáticamente:

### Tablas Principales
- `users` - Perfiles de usuarios
- `categories` - Categorías de servicios
- `pitutos` - Trabajos/servicios
- `applications` - Aplicaciones a trabajos
- `reviews` - Reseñas y calificaciones

### Datos Iniciales
- 12 categorías por defecto (Limpieza, Jardinería, Plomería, etc.)
- Usuario de prueba: gabriel@pantoja.cl
- Ejemplos de pitutos

## 🔐 Autenticación

### JWT Configuration
- **ANON_KEY**: Para acceso público (frontend)
- **SERVICE_ROLE_KEY**: Para operaciones administrativas
- **JWT_SECRET**: Para firma de tokens

### Configuración de Auth
- Signup habilitado
- Email verification opcional
- Persistent sessions
- Realtime habilitado

## 🔧 **Comandos de Mantenimiento**

### **Verificar Estado de Servicios:**
```bash
# Estado de Nginx
sudo systemctl status nginx

# Estado de contenedores Docker
docker ps

# Ver logs de aplicación  
docker-compose logs -f

# Ver logs de Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

### **Gestión SSL:**
```bash
# Renovar certificados manualmente
sudo certbot renew
sudo systemctl reload nginx

# Verificar estado de renovación automática
sudo systemctl status snap.certbot.renew.timer
```

### **Reiniciar Servicios:**
```bash
# Reiniciar Nginx
sudo systemctl restart nginx

# Reiniciar contenedores Docker
cd /home/gabriel/apps/pitutito/
docker-compose restart

# Recargar configuración de Nginx
sudo systemctl reload nginx
```

### **Comandos Remotos (desde local):**
```bash
# Ver estado remoto
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/pitutito && sudo docker compose -f docker-compose.production.yml ps'

# Ver logs remotos  
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/pitutito && sudo docker compose -f docker-compose.production.yml logs -f'

# Backup de base de datos
ssh gabriel@167.172.251.27 'sudo docker compose -f /home/gabriel/apps/pitutito/docker-compose.production.yml exec pitutito-postgres pg_dump -U postgres pitutito_trabajos_db > pitutito_backup_$(date +%Y%m%d).sql'
```

## 📱 Variables de Entorno del Frontend

El frontend usa estas variables (ya configuradas):
```bash
VITE_SUPABASE_URL=http://167.172.251.27:8001
VITE_SUPABASE_ANON_KEY=<anon-key-generado>
```

## ⚠️ **Problemas Comunes y Soluciones**

### **1. Puerto 80 no responde**
```bash
# Verificar que el sitio esté habilitado
ls -la /etc/nginx/sites-enabled/
# Debe aparecer: pitutito.cl -> ../sites-available/pitutito.cl

# Si no existe el enlace
cd /etc/nginx/sites-enabled/
sudo ln -sf ../sites-available/pitutito.cl .
sudo systemctl reload nginx
```

### **2. SSL falla con Let's Encrypt**  
```bash
# Verificar firewall
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Verificar que dominios apunten al VPS  
nslookup pitutito.cl  # Debe mostrar 167.172.251.27

# Reintentar certificados
sudo certbot --nginx -d pitutito.cl -d www.pitutito.cl -d studio.pitutito.cl
```

### **3. Aplicación no carga**
```bash
# Verificar contenedores Docker
docker ps

# Verificar que puerto 3000 esté ocupado  
sudo ss -tlnp | grep :3000

# Reiniciar contenedores si es necesario
cd /home/gabriel/apps/pitutito/
docker-compose restart
```

### **4. Nginx no inicia**
```bash
# Verificar configuración
sudo nginx -t

# Ver logs de error
sudo tail -20 /var/log/nginx/error.log

# Restaurar configuración si hay errores
sudo cp /etc/nginx/nginx.conf.backup /etc/nginx/nginx.conf
sudo systemctl restart nginx
```

## 🚨 **Información de Emergencia**

### **Acceso al VPS:**
```bash
ssh gabriel@167.172.251.27
```

### **Restaurar configuración:**
```bash
# Restaurar Nginx a configuración original
sudo cp /etc/nginx/nginx.conf.backup /etc/nginx/nginx.conf
sudo systemctl restart nginx
```

### **Logs de diagnóstico:**
```bash
# Logs de Nginx
sudo tail -100 /var/log/nginx/error.log

# Logs de SSL
sudo tail -100 /var/log/letsencrypt/letsencrypt.log

# Logs de contenedores
cd /home/gabriel/apps/pitutito/
docker-compose logs --tail=100
```

## 🎯 **Checklist de Verificación**

### **✅ Pre-Producción:**
- [x] DNS configurado en Cloudflare (pitutito.cl → 167.172.251.27)
- [x] VPS configurado (Ubuntu 24.04.1 LTS)
- [x] Docker y Docker Compose instalados
- [x] Aplicación React funcionando en contenedor (puerto 3000)
- [x] Supabase autoalojado funcionando (puerto 8005)

### **✅ Configuración Nginx:**
- [x] Nginx instalado y corriendo
- [x] Configuración `/etc/nginx/sites-available/pitutito.cl` creada
- [x] Enlace simbólico en `/etc/nginx/sites-enabled/` activo
- [x] Proxy hacia puertos 3000 y 8005 configurado
- [x] Puerto 80 respondiendo correctamente

### **✅ SSL/HTTPS:**
- [x] Firewall configurado (puertos 80, 443 abiertos)
- [x] Certbot instalado vía snap
- [x] Certificados SSL obtenidos para 3 dominios
- [x] HTTPS funcionando en producción
- [x] Renovación automática configurada

### **✅ Verificación Final:**
- [x] https://pitutito.cl carga la aplicación React
- [x] https://studio.pitutito.cl accede a Supabase Studio  
- [x] Todos los assets estáticos se cargan correctamente
- [x] Sin errores 404 o 500 en logs de Nginx

---

## 🎉 **Estado Final: ÉXITO COMPLETO - Agosto 2025**

✅ **Aplicación React desplegada y funcionando**  
✅ **HTTPS configurado con renovación automática**  
✅ **Proxy Nginx configurado correctamente**  
✅ **Firewall configurado y seguro**  
✅ **DNS funcionando correctamente**  
✅ **Supabase Studio accesible**  

**🌐 pitutito.cl está oficialmente LIVE en producción con HTTPS!**

---

**Desplegado:** 6 de agosto de 2025  
**Tiempo total:** ~3 horas de configuración  
**Estado:** Producción estable  
**Próxima revisión:** Renovación SSL automática en 90 días  
**Contacto:** gabriel@pantoja.cl
