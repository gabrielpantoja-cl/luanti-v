# 🌐 Configuración de Dominio pitutito.cl con Cloudflare

## Resumen de Cambios - Agosto 2025

### ✅ Archivos Creados/Actualizados

**Configuración Nginx para Cloudflare:**
- `nginx-cloudflare.conf` - Nginx optimizado para Cloudflare con:
  - Real IP detection de Cloudflare
  - Configuración SSL/TLS transparente
  - Rate limiting optimizado
  - CORS headers para API
  - Subdominios: pitutito.cl, api.pitutito.cl, studio.pitutito.cl

**Scripts de Deployment:**
- `setup-cloudflare-domain.sh` - Guía interactiva para configurar DNS en Cloudflare
- `deploy-with-domain.sh` - Deployment completo con dominio personalizado
- `nginx-production.conf` - Configuración Nginx tradicional (Let's Encrypt)
- `setup-ssl.sh` - Configuración SSL con Let's Encrypt (alternativa)

**Variables de Entorno Actualizadas:**
- `.env.production` - URLs actualizadas para usar dominios
- `.env` - Frontend configurado para usar api.pitutito.cl

## 🏗️ Arquitectura Final

```
┌─────────────────────────────────────────────────────────┐
│                    CLOUDFLARE CDN                       │
│  🌤️  SSL/TLS, DDoS Protection, Caching, Analytics     │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────┐
│              VPS (167.172.251.27)                      │
│                     │                                   │
│  ┌──────────────────▼──────────────────┐               │
│  │         NGINX REVERSE PROXY         │               │
│  │  🌐 pitutito.cl → :3000            │               │
│  │  🔌 api.pitutito.cl → :8001        │               │
│  │  📊 studio.pitutito.cl → :8005     │               │
│  └─────────────────┬───────────────────┘               │
│                    │                                   │
│  ┌─────────────────┼───────────────────────────────────┤
│  │     DOCKER CONTAINERS                               │
│  │                 │                                   │
│  │  📱 React App (:3000)                              │
│  │  🔌 Kong API Gateway (:8001)                       │
│  │  📊 Supabase Studio (:8005)                        │
│  │  🗄️  PostgreSQL (:54324)                           │
│  │  🔐 Auth + Storage + Realtime                      │
│  └─────────────────────────────────────────────────────┘
└─────────────────────────────────────────────────────────┘
```

## 🚀 Proceso de Deployment

### Paso 1: Configuración DNS en Cloudflare
```bash
./setup-cloudflare-domain.sh
```

**Registros DNS requeridos:**
| Tipo | Nombre | Contenido | Proxy |
|------|--------|-----------|--------|
| A | @ | 167.172.251.27 | ✅ |
| A | www | 167.172.251.27 | ✅ |
| A | api | 167.172.251.27 | ✅ |
| A | studio | 167.172.251.27 | ✅ |

### Paso 2: Deployment Completo
```bash
./deploy-with-domain.sh
```

**Acciones automáticas:**
1. ✅ Instala Nginx en VPS
2. ✅ Configura reverse proxy para subdominios
3. ✅ Configura firewall (UFW)
4. ✅ Actualiza variables de entorno para dominios
5. ✅ Re-despliega aplicación con nueva configuración
6. ✅ Verifica conectividad

## 🔧 Configuración de Cloudflare

### SSL/TLS Settings
- **Encryption Mode**: Full (Strict)
- **Always Use HTTPS**: ✅ Enabled
- **HSTS**: ✅ Enabled
- **Minimum TLS Version**: 1.2

### Page Rules (Recomendadas)
1. `www.pitutito.cl/*` → Redirect to `https://pitutito.cl/$1` (301)
2. `http://pitutito.cl/*` → Always Use HTTPS

### Speed Optimizations
- **Auto Minify**: HTML, CSS, JavaScript
- **Brotli**: Enabled
- **Early Hints**: Enabled
- **HTTP/3**: Enabled

## 🌐 URLs Finales

### Producción (Cloudflare)
- **🏠 Sitio Principal**: https://pitutito.cl
- **🌍 Con WWW**: https://www.pitutito.cl (redirige a pitutito.cl)
- **🔌 API**: https://api.pitutito.cl
- **📊 Studio**: https://studio.pitutito.cl

### Desarrollo (Acceso Directo)
- **📱 App**: http://167.172.251.27:3000
- **🔌 API**: http://167.172.251.27:8001
- **📊 Studio**: http://167.172.251.27:8005
- **🗄️ PostgreSQL**: 167.172.251.27:54324

## 📊 Monitoreo y Logs

### Nginx Logs
```bash
ssh gabriel@167.172.251.27 'sudo tail -f /var/log/nginx/access.log'
ssh gabriel@167.172.251.27 'sudo tail -f /var/log/nginx/error.log'
```

### Docker Containers
```bash
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/pitutito && docker compose -f docker-compose.production.yml ps'
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/pitutito && docker compose -f docker-compose.production.yml logs -f'
```

### Cloudflare Analytics
- Ve al Dashboard de Cloudflare > Analytics & Logs
- Traffic overview, Security events, Performance insights

## 🔒 Seguridad Implementada

### Cloudflare
- ✅ DDoS Protection
- ✅ Web Application Firewall (WAF)
- ✅ SSL/TLS Encryption
- ✅ Bot Fight Mode
- ✅ Rate Limiting

### VPS (UFW Firewall)
```bash
# Puertos abiertos:
22    - SSH
80    - HTTP (Nginx)
443   - HTTPS (futuro)
3000  - React App (desarrollo)
8001  - Kong API (desarrollo)
8005  - Supabase Studio (desarrollo)
54324 - PostgreSQL (desarrollo)
```

### Nginx
- ✅ Rate limiting por IP
- ✅ Security headers
- ✅ Real IP detection (Cloudflare)
- ✅ Access logs con geolocalización

## 🔧 Comandos Útiles

### Restart Services
```bash
# Reiniciar Nginx
ssh gabriel@167.172.251.27 'sudo systemctl restart nginx'

# Reiniciar Docker containers
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/pitutito && docker compose -f docker-compose.production.yml restart'
```

### Update Application
```bash
# Re-deploy completo
./deploy-with-domain.sh

# Solo actualizar código frontend
./deploy-no-sudo.sh
```

### Check DNS Propagation
- https://dnschecker.org
- `nslookup pitutito.cl`
- `dig pitutito.cl`

## 🚨 Troubleshooting

### Dominio no resuelve
1. Verificar configuración DNS en Cloudflare
2. Esperar propagación (hasta 48h)
3. Verificar con `dnschecker.org`

### Error 502 Bad Gateway
1. Verificar que Docker containers estén running
2. Verificar logs de Nginx
3. Verificar conectividad interna

### SSL/HTTPS issues
1. Verificar configuración SSL en Cloudflare
2. Asegurar que modo sea "Full (Strict)"
3. Verificar certificados en Cloudflare

## 📈 Métricas de Performance

### Cloudflare Analytics
- **Requests**: Monitorear tráfico total
- **Bandwidth**: Uso de ancho de banda
- **Threats**: Ataques bloqueados
- **Performance**: Core Web Vitals

### Docker Resources
```bash
# Uso de recursos
ssh gabriel@167.172.251.27 'docker stats'

# Espacio en disco
ssh gabriel@167.172.251.27 'df -h'
```

---

**Última actualización**: Agosto 2025  
**Versión**: 2.0 (Con dominio personalizado)  
**Estado**: ✅ Producción con Cloudflare