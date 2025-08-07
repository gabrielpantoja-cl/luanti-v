# 🛠️ Scripts de Configuración y Mantenimiento

## 📋 Resumen

Después de implementar GitHub Actions para despliegue automatizado, estos son los únicos scripts que se mantienen en el proyecto. Cada uno tiene un propósito específico para configuración inicial o mantenimiento del servidor.

## 🚀 Scripts Disponibles

### 🔑 **setup-ssh-key.sh**
```bash
./setup-ssh-key.sh
```
**Propósito**: Generar y configurar claves SSH para acceso automatizado al VPS
- Genera pares de claves SSH
- Configura acceso sin contraseña al servidor
- Útil para configuración inicial o rotación de claves

**Cuándo usar**: 
- Primera configuración del VPS
- Rotación de claves SSH por seguridad
- Troubleshooting de acceso SSH

---

### 🐳 **setup-docker-permissions.sh**
```bash
./setup-docker-permissions.sh
```
**Propósito**: Configurar permisos de Docker para evitar usar sudo
- Añade usuario al grupo docker
- Configura permisos correctos
- Permite que GitHub Actions funcione sin sudo

**Cuándo usar**:
- Después de instalar Docker en el VPS
- Si GitHub Actions falla por permisos de Docker
- Configuración inicial del servidor

---

### 🔐 **setup-keys.sh**
```bash
./setup-keys.sh
```
**Propósito**: Generar JWT secrets y API keys para Supabase
- Genera JWT_SECRET seguro (256 caracteres)
- Crea ANON_KEY y SERVICE_ROLE_KEY
- Configuración de autenticación de Supabase

**Cuándo usar**:
- Primera configuración de Supabase
- Rotación de keys por seguridad
- Regenerar keys comprometidas

---

### ☁️ **setup-cloudflare-domain.sh**
```bash
./setup-cloudflare-domain.sh
```
**Propósito**: Guía interactiva para configurar DNS en Cloudflare
- Configuración de registros A/CNAME
- Configuración de proxy de Cloudflare
- Optimización de DNS para el dominio

**Cuándo usar**:
- Configuración inicial del dominio
- Migración de dominios
- Cambio de proveedor DNS

---

### 🌐 **setup-nginx-domain.sh**
```bash
./setup-nginx-domain.sh
```
**Propósito**: Instalar y configurar Nginx para el dominio
- Instalación de Nginx
- Configuración de virtual hosts
- Setup de proxy reverso para la aplicación

**Cuándo usar**:
- Primera configuración del servidor web
- Cambio de dominio
- Reconfiguración de Nginx

---

### 🔒 **setup-ssl.sh**
```bash
./setup-ssl.sh
```
**Propósito**: Configurar HTTPS con certificados Let's Encrypt
- Instalación de Certbot
- Obtención de certificados SSL
- Configuración de renovación automática

**Cuándo usar**:
- Activar HTTPS por primera vez
- Renovación manual de certificados
- Troubleshooting de SSL

---

## ⚡ **¿Qué pasó con los scripts de deployment?**

Los siguientes scripts fueron **removidos** porque ahora usamos **GitHub Actions**:

~~deploy-automated.sh~~ ❌ **Removido** - Reemplazado por `.github/workflows/deploy.yml`  
~~deploy-interactive.sh~~ ❌ **Removido** - Reemplazado por GitHub Actions  
~~deploy-no-sudo.sh~~ ❌ **Removido** - Reemplazado por GitHub Actions  
~~deploy-to-vps.sh~~ ❌ **Removido** - Reemplazado por GitHub Actions  
~~deploy-with-domain.sh~~ ❌ **Removido** - Reemplazado por GitHub Actions  
~~deploy-with-prompt.sh~~ ❌ **Removido** - Reemplazado por GitHub Actions  

### 🎯 **Despliegue Actual**

**✅ Método actual**: Push a `main` → GitHub Actions automáticamente despliega  
**📍 Workflow**: `.github/workflows/deploy.yml`  
**🔗 Monitoreo**: GitHub Actions tab en el repositorio

---

## 🚀 **Flujo de Configuración Inicial**

Para un VPS nuevo, ejecuta los scripts en este orden:

```bash
# 1. Configurar acceso SSH
./setup-ssh-key.sh

# 2. Configurar permisos de Docker  
./setup-docker-permissions.sh

# 3. Generar keys de Supabase
./setup-keys.sh

# 4. Configurar dominio (si es necesario)
./setup-cloudflare-domain.sh

# 5. Configurar Nginx
./setup-nginx-domain.sh

# 6. Activar HTTPS
./setup-ssl.sh
```

**Después de esto**, el despliegue se hace automáticamente con GitHub Actions.

---

## 🔧 **Troubleshooting Common Issues**

### SSH no funciona
```bash
./setup-ssh-key.sh
```

### Docker requiere sudo
```bash
./setup-docker-permissions.sh
```

### Problemas de autenticación en Supabase
```bash
./setup-keys.sh
```

### SSL expirado o problemático
```bash
./setup-ssl.sh
```

---

## 📊 **Scripts Mantenidos vs Removidos**

| Estado | Cantidad | Propósito |
|--------|----------|-----------|
| ✅ **Mantenidos** | 6 | Setup y mantenimiento |
| ❌ **Removidos** | 9 | Deployment (reemplazado por GitHub Actions) |

**Total reducción**: De 15 scripts a 6 scripts (60% de reducción) 🎉

---

## 📞 **Soporte**

Si tienes problemas con algún script:
1. Revisa que tienes permisos de ejecución: `chmod +x script.sh`
2. Ejecuta con bash si es necesario: `bash script.sh`
3. Contacta: gabriel@pantoja.cl