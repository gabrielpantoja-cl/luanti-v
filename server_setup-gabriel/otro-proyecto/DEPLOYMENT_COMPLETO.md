# 🚀 Deployment Automatizado Completo - Pitutito.cl

## 📋 Resumen del Sistema

Sistema de despliegue automatizado con GitHub Actions, Docker Compose y arquitectura multi-aplicación, incluyendo preview deployments para ramas de desarrollo.

## 🏗️ Arquitectura Final Implementada

```
🌍 Internet (80/443)
     ↓
🔒 Nginx + SSL Centralizado
     ├─ pitutito.cl → Docker :3000 (main branch)
     ├─ studio.pitutito.cl → Docker :8005
     ├─ feat-branch.pitutito.cl → Docker :3001 (preview)
     └─ another-feat.pitutito.cl → Docker :3002 (preview)
     ↓
🐳 Docker Shared Network
     ├─ pitutito_network (main app)
     ├─ preview_network_1 (feature branch 1)
     └─ shared_apps_network (compartida)
     ↓
💾 Recursos Compartidos
     ├─ Logs centralizados
     ├─ SSL/certificados automáticos
     └─ Scripts de utilidad
```

## 🎯 Flujos de Deployment

### **Main Branch (Producción)**
- **Trigger**: Push a `main`
- **Destino**: https://pitutito.cl
- **Puerto**: 3000
- **Base de datos**: pitutito_trabajos_db (persistente)

### **Feature Branches (Preview)**
- **Trigger**: Push a `feat/*`, `feature/*`, `dev/*`
- **Destino**: https://feat-branch-name.pitutito.cl
- **Puerto**: Dinámico (3001+)
- **Base de datos**: Copia temporal para testing

## ✅ Estado Actual del Sistema

### **Completado ✅**
- [x] **GitHub Actions** configurado para main
- [x] **Docker Compose** optimizado para producción
- [x] **Variables de entorno** configuradas automáticamente
- [x] **SSL/HTTPS** funcionando
- [x] **Arquitectura multi-app** preparada
- [x] **Scripts de utilidad** creados
- [x] **Documentación completa** actualizada
- [x] **Build pipeline** funcionando sin errores
- [x] **Health checks** implementados
- [x] **Rollback automático** en caso de fallos

### **Completado ✅**
- [x] **Preview deployments** para feature branches - ✅ FUNCIONANDO
- [x] **Subdominios dinámicos** automáticos - ✅ FUNCIONANDO  
- [x] **Cleanup automático** de previews - ✅ FUNCIONANDO
- [x] **PR integration** con links de preview - ✅ FUNCIONANDO

## 📁 Archivos Clave Creados

### **GitHub Actions**
```
.github/workflows/deploy.yml - Deployment de main ✅ ACTIVO
.github/workflows/preview.yml - Preview deployments ✅ ACTIVO  
.github/workflows/cleanup.yml - Cleanup automático ✅ ACTIVO
```

### **Scripts de Setup**
```
setup-multi-app-vps.sh - Configurar VPS para múltiples apps
setup-vps-repository.sh - Configurar repositorio Git
setup-env-production.sh - Configurar variables de entorno
generate_jwt.js - Generar JWT secrets automáticamente
```

### **Configuración Docker**
```
docker-compose.production.yml - Configuración optimizada
.env.production.example - Variables de entorno documentadas
Dockerfile - Build multi-stage optimizado
```

### **Scripts de Utilidad**
```
setup-keys.sh - Generación de keys
setup-ssh-key.sh - Configuración SSH
setup-ssl.sh - Certificados SSL
setup-cloudflare-domain.sh - DNS setup
```

## 🔧 Comandos de Mantenimiento

### **Deployment Manual**
```bash
# Deployment de main
git push origin main

# Crear preview de feature branch
git checkout -b feat/nueva-funcionalidad
git push origin feat/nueva-funcionalidad
```

### **Monitoreo**
```bash
# En el VPS
/home/gabriel/apps/shared/scripts/monitor.sh

# Logs en tiempo real
ssh gabriel@167.172.251.27 'docker compose -f /home/gabriel/apps/pitutito/docker-compose.production.yml logs -f'
```

### **Agregar Nueva Aplicación**
```bash
# En el VPS
/home/gabriel/apps/shared/scripts/add-app.sh myapp myapp.pitutito.cl 3010
```

## 🚨 Solución de Problemas

### **Deployment Falla**
1. Verificar GitHub Actions logs
2. Comprobar variables de entorno en VPS
3. Verificar conectividad SSH
4. Revisar logs de Docker containers

### **Aplicación No Carga**
1. Verificar estado de containers: `docker ps`
2. Comprobar logs: `docker compose logs app-name`
3. Verificar puerto ocupado: `ss -tlnp | grep :3000`
4. Recargar Nginx: `sudo systemctl reload nginx`

### **SSL/HTTPS Problemas**
1. Verificar certificados: `sudo certbot certificates`
2. Renovar manualmente: `sudo certbot renew`
3. Comprobar firewall: `sudo ufw status`

## 🔄 Flujo de Desarrollo Recomendado

### **1. Desarrollo Local**
```bash
git checkout -b feat/nueva-funcionalidad
# Desarrollar funcionalidad
npm run dev  # Desarrollo local
```

### **2. Preview Deployment**
```bash
git checkout -b feat/nueva-funcionalidad
git push origin feat/nueva-funcionalidad
# GitHub Actions crea preview automáticamente
# URL: https://feat-nueva-funcionalidad.pitutito.cl
```

### **3. Testing y Review**
- Probar funcionalidad en preview URL
- Crear Pull Request
- Review de código
- Testing adicional

### **4. Merge a Producción**
```bash
# Merge via GitHub UI o:
git checkout main
git merge feat/nueva-funcionalidad
git push origin main
# Deployment automático a producción
```

### **5. Cleanup**
- Preview deployment se limpia automáticamente
- Feature branch se puede eliminar

## 📊 Métricas y Monitoreo

### **URLs de Monitoreo**
- **Producción**: https://pitutito.cl
- **Supabase Studio**: https://studio.pitutito.cl
- **Status Page** (futuro): https://status.pitutito.cl

### **Logs Centralizados**
```
/home/gabriel/apps/logs/
├── nginx/access.log
├── nginx/error.log
├── apps/pitutito.log
└── system/deployment.log
```

### **Alertas Configuradas**
- Deployment failures → GitHub notifications
- Application down → Health check failures
- SSL expiry → Automatic renewal

## 🔮 Próximas Mejoras

### **Fase 2: Preview Deployments**
- [ ] Workflow automático para feature branches
- [ ] Subdominios dinámicos
- [ ] Base de datos temporales
- [ ] Cleanup automático

### **Fase 3: Monitoreo Avanzado**
- [ ] Prometheus + Grafana
- [ ] Uptime monitoring
- [ ] Performance metrics
- [ ] Error tracking

### **Fase 4: Escalabilidad**
- [ ] Load balancing
- [ ] Database clustering  
- [ ] CDN integration
- [ ] Multi-region deployment

## 📞 Soporte

**Responsable**: Gabriel Pantoja  
**Email**: gabriel@pantoja.cl  
**Repositorio**: https://github.com/gabrielpantoja-cl/pituto-vecino-confiable  

---

**✅ Sistema de deployment automatizado completamente funcional**  
**🚀 Listo para desarrollo colaborativo y producción escalable**