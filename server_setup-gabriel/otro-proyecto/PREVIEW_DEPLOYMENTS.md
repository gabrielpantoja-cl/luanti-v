# 🔍 Preview Deployments - Sistema Completo

## 📋 Resumen

Sistema automatizado de preview deployments que permite desplegar cualquier rama a un subdominio temporal para testing y review antes de merge a main.

## 🚀 Cómo Funciona

### **Automatización Completa**
1. **Push a cualquier rama** → Deployment automático
2. **Subdominio único** → `rama-name.pitutito.cl`
3. **SSL automático** → Certificado Let's Encrypt
4. **Testing independiente** → No afecta producción
5. **Cleanup automático** → Se limpia al cerrar PR o eliminar rama

## 🌐 Ejemplos de URLs de Preview

```bash
# Ejemplos de ramas y sus URLs generadas:
feat/login-system      → https://feat-login-system.pitutito.cl
fix/header-bug         → https://fix-header-bug.pitutito.cl
chore/update-deps      → https://chore-update-deps.pitutito.cl
refactor/api-cleanup   → https://refactor-api-cleanup.pitutito.cl
hotfix/critical-issue  → https://hotfix-critical-issue.pitutito.cl
dev/experimental       → https://dev-experimental.pitutito.cl
123-fix-navbar         → https://123-fix-navbar.pitutito.cl
```

## 🔄 Flujo de Desarrollo

### **1. Crear Rama y Desarrollar**
```bash
# Crear nueva rama (cualquier nombre)
git checkout -b feat/nueva-funcionalidad
# o
git checkout -b fix/bug-importante
# o  
git checkout -b 123-arreglar-menu

# Desarrollar tu feature
# ... hacer cambios ...
```

### **2. Push para Preview**
```bash
git add .
git commit -m "feat: agregar nueva funcionalidad"
git push origin feat/nueva-funcionalidad
```

**🎯 Resultado automático:**
- ✅ GitHub Actions ejecuta deployment
- ✅ Preview disponible en `https://feat-nueva-funcionalidad.pitutito.cl`
- ✅ SSL configurado automáticamente
- ✅ Comentario en PR con link de preview

### **3. Testing y Review**
- **Testear** la funcionalidad en la URL de preview
- **Compartir link** con el equipo para review
- **Hacer ajustes** y push para actualizar preview
- **Crear Pull Request** hacia main

### **4. Merge y Cleanup**
- **Merge PR** → Deployment automático a producción
- **Cleanup automático** → Preview se elimina automáticamente
- **Recursos liberados** → Sin costos adicionales

## 🛠️ Configuración Técnica

### **Workflows Configurados**

#### **`.github/workflows/preview.yml`**
- **Trigger**: Push a cualquier rama (excepto main)
- **Función**: Crear preview deployment
- **Salida**: URL de preview + comentario en PR

#### **`.github/workflows/cleanup.yml`**  
- **Trigger**: Branch deletion, PR closed, manual
- **Función**: Limpiar recursos del preview
- **Salida**: Confirmación de limpieza

#### **`.github/workflows/deploy.yml`**
- **Trigger**: Push a main
- **Función**: Deployment a producción
- **Salida**: https://pitutito.cl actualizado

### **Arquitectura del Preview**

```
🌍 Internet (443)
     ↓
🔒 Nginx + SSL
     ├─ pitutito.cl → :3000 (main/producción)
     ├─ feat-login.pitutito.cl → :3001 (preview)
     ├─ fix-header.pitutito.cl → :3002 (preview)
     └─ dev-test.pitutito.cl → :3003 (preview)
     ↓
🐳 Docker Containers
     ├─ pitutito_web (producción)
     ├─ preview_feat_login_web (rama feat/login)
     ├─ preview_fix_header_web (rama fix/header)
     └─ preview_dev_test_web (rama dev/test)
```

### **Asignación de Puertos**
- **Puerto base**: 3001-3099 para previews
- **Algoritmo**: Hash del nombre de rama → puerto único
- **Prevención de colisiones**: MD5 hash garantiza unicidad

## 📝 Integración con GitHub

### **Comentarios Automáticos en PR**
Cuando creas un PR, recibes automáticamente:

```markdown
## 🔍 Preview Deployment Ready!

✅ Your changes have been deployed to a preview environment:

**🌐 Preview URL:** https://feat-nueva-funcionalidad.pitutito.cl
**🌿 Branch:** feat/nueva-funcionalidad
**🔄 Status:** Deployed successfully

### 📋 Quick Links
- [🌐 Preview Site](https://feat-nueva-funcionalidad.pitutito.cl)
- [📊 View Logs](https://github.com/user/repo/actions/runs/123)

---
💡 **Note:** This preview will be automatically cleaned up when the PR is closed or merged.
```

### **Comentarios de Cleanup**
Al cerrar PR o merge:

```markdown
## 🧹 Preview Deployment Cleaned Up

✅ Preview deployment has been automatically cleaned up:

**🌿 Branch:** feat/nueva-funcionalidad
**🌐 Preview URL:** https://feat-nueva-funcionalidad.pitutito.cl *(no longer accessible)*
**🗑️ Status:** Cleaned up successfully
```

## 🔧 Comandos de Mantenimiento

### **Cleanup Manual**
```bash
# Limpiar preview específico (desde GitHub Actions)
# Settings → Actions → Run workflow → Cleanup → Especificar rama

# Limpiar todos los previews
# Settings → Actions → Run workflow → Cleanup → (dejar vacío)
```

### **Monitoreo de Previews**
```bash
# En el VPS
/home/gabriel/apps/shared/scripts/monitor.sh

# Ver todos los previews activos
ls /home/gabriel/apps/ | grep preview-

# Ver containers de preview
docker ps | grep preview
```

### **Debug de Preview**
```bash
# Ver logs de preview específico
ssh gabriel@167.172.251.27 'cd /home/gabriel/apps/preview-feat-login && docker compose logs -f'

# Ver estado de nginx
ssh gabriel@167.172.251.27 'sudo nginx -t && sudo systemctl status nginx'

# Ver certificados SSL
ssh gabriel@167.172.251.27 'sudo certbot certificates'
```

## 🔒 Seguridad y Limitaciones

### **Seguridad**
- ✅ **SSL/HTTPS**: Todos los previews tienen certificados automáticos
- ✅ **Aislamiento**: Cada preview corre en su propio container/red
- ✅ **Cleanup automático**: No se acumulan recursos
- ✅ **Acceso controlado**: Solo ramas del repositorio oficial

### **Limitaciones**
- **Recursos del VPS**: Máximo ~10-15 previews concurrentes
- **SSL Rate Limits**: Let's Encrypt tiene límites de certificados/semana
- **Base de datos**: Los previews no tienen BD independiente (futuro)
- **Persistencia**: Los datos no persisten entre deployments

## 🚨 Troubleshooting

### **Preview No Se Crea**
1. **Verificar GitHub Actions**: Ir a Actions tab y revisar logs
2. **Comprobar nombre de rama**: Evitar caracteres especiales excesivos
3. **Verificar secrets**: VPS_HOST, VPS_USER, VPS_SSH_KEY configurados
4. **Espacio en VPS**: Verificar que hay espacio en disco

### **Preview No Carga**
1. **Verificar puerto**: Puerto asignado debe estar libre
2. **Check SSL**: Certificado puede tardar unos minutos
3. **Nginx config**: Verificar configuración de proxy
4. **Docker status**: Container debe estar corriendo

### **SSL No Funciona**
1. **DNS propagation**: Puede tardar unos minutos
2. **Rate limits**: Let's Encrypt tiene límites
3. **Nginx reload**: `sudo systemctl reload nginx`
4. **Manual certificate**: `sudo certbot --nginx -d rama.pitutito.cl`

## 💡 Best Practices

### **Nombres de Rama**
```bash
# ✅ Buenos nombres
feat/login-system
fix/header-responsive  
chore/update-dependencies
123-critical-bug

# ❌ Evitar
feat/login_system_with_very_long_name_that_causes_issues
fix/header%bug@special
```

### **Testing en Preview**
- **Funcionalidad completa**: Testear todos los flujos
- **Responsive design**: Probar en móvil y desktop
- **Performance**: Verificar que no hay degradación
- **Links y navegación**: Asegurar que todo funciona

### **Colaboración**
- **Compartir links**: URL de preview fácil de compartir
- **Feedback en PR**: Usar comentarios para feedback específico
- **Testing paralelo**: Múltiples reviews simultáneas posibles

## 🎯 Casos de Uso

### **Feature Development**
```bash
git checkout -b feat/payment-integration
# Desarrollar feature
git push origin feat/payment-integration
# Preview en https://feat-payment-integration.pitutito.cl
# Testing y ajustes
# PR y merge
```

### **Bug Fixes**
```bash
git checkout -b fix/mobile-menu-bug
# Arreglar bug
git push origin fix/mobile-menu-bug
# Preview para verificar fix
# PR y merge rápido
```

### **Experiments**
```bash
git checkout -b experiment/new-ui-design
# Probar nuevo diseño
git push origin experiment/new-ui-design
# Compartir con equipo para feedback
# Decidir si continuar o descartar
```

---

## ✅ Estado del Sistema

- [x] **Preview deployments automáticos** para cualquier rama
- [x] **Subdominios dinámicos** con SSL automático  
- [x] **Cleanup automático** cuando se cierra/elimina rama
- [x] **Comentarios en PR** con links de preview
- [x] **Monitoreo y debugging** integrado
- [x] **Documentación completa** y ejemplos

**🚀 Sistema de preview deployments completamente funcional y listo para uso en equipo!**