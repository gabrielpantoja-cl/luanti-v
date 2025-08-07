# 🚀 Configuración de GitHub Actions para Despliegue Automatizado

## 📋 Resumen

Esta guía explica cómo configurar el despliegue automatizado de **Pitutito.cl** utilizando GitHub Actions. El sistema desplegará automáticamente los cambios de la rama `main` a tu VPS en producción.

## 🔧 Requisitos Previos

- ✅ VPS configurado y funcionando con la aplicación
- ✅ Acceso SSH al VPS
- ✅ Aplicación funcionando manualmente en el VPS
- ✅ Git configurado en el VPS con el repositorio clonado

## 🗝️ Configuración de GitHub Secrets

### 1. Acceder a la Configuración de Secrets

1. Ve a tu repositorio en GitHub
2. Haz clic en **Settings** (configuración)
3. En el menú lateral, selecciona **Secrets and variables** > **Actions**
4. Haz clic en **New repository secret**

### 2. Crear los Secrets Necesarios

Necesitas crear estos 3 secrets:

#### `VPS_HOST`
```
Valor: 167.172.251.27
Descripción: IP del VPS donde está alojada la aplicación
```

#### `VPS_USER`
```
Valor: gabriel
Descripción: Usuario SSH para conectarse al VPS
```

#### `VPS_SSH_KEY`
```
Valor: [contenido de tu clave SSH privada]
Descripción: Clave privada SSH para autenticación
```

## 🔑 Generar y Configurar Claves SSH

### Paso 1: Generar Clave SSH (en tu máquina local)

```bash
# Generar nueva clave SSH específica para GitHub Actions
ssh-keygen -t ed25519 -C "github-actions@pitutito.cl" -f ~/.ssh/pitutito_deploy

# Esto creará dos archivos:
# ~/.ssh/pitutito_deploy     (clave privada - para GitHub Secret)
# ~/.ssh/pitutito_deploy.pub (clave pública - para el VPS)
```

### Paso 2: Agregar Clave Pública al VPS

```bash
# Copiar clave pública al VPS
ssh-copy-id -i ~/.ssh/pitutito_deploy.pub gabriel@167.172.251.27

# O manualmente:
cat ~/.ssh/pitutito_deploy.pub | ssh gabriel@167.172.251.27 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Paso 3: Probar Conexión

```bash
# Probar conexión con la nueva clave
ssh -i ~/.ssh/pitutito_deploy gabriel@167.172.251.27 "echo 'Conexión exitosa'"
```

### Paso 4: Configurar el Secret VPS_SSH_KEY

```bash
# Mostrar contenido de la clave privada
cat ~/.ssh/pitutito_deploy
```

Copia **todo el contenido** (incluyendo `-----BEGIN OPENSSH PRIVATE KEY-----` y `-----END OPENSSH PRIVATE KEY-----`) y pégalo en el secret `VPS_SSH_KEY`.

## 📁 Preparar el VPS

### 1. Configurar el repositorio en el VPS (AUTOMATIZADO)

**Opción A: Usar script automatizado (RECOMENDADO)**
```bash
# Ejecutar desde tu máquina local
./setup-vps-repository.sh
```

Este script:
- ✅ Verifica si el directorio existe
- ✅ Clona el repositorio si no existe
- ✅ Hace backup de archivos de configuración existentes
- ✅ Configura el repositorio Git correctamente
- ✅ Verifica que todo esté funcionando

**Opción B: Configuración manual**
```bash
# Conectarse al VPS
ssh gabriel@167.172.251.27

# Si el directorio no existe o no es un repo git
sudo rm -rf /home/gabriel/apps/pitutito
git clone https://github.com/gabrielpantoja-cl/pituto-vecino-confiable.git /home/gabriel/apps/pitutito
cd /home/gabriel/apps/pitutito

# Verificar que esté conectado al repositorio
git remote -v
# Debe mostrar: https://github.com/gabrielpantoja-cl/pituto-vecino-confiable.git
```

### 2. Configurar permisos de Docker (si no está hecho)

```bash
# En el VPS, asegurar que el usuario pueda usar docker sin sudo
sudo usermod -aG docker gabriel

# Reiniciar sesión o ejecutar:
newgrp docker

# Verificar que funciona
docker ps
```

## 🔄 Cómo Funciona el Workflow

### Trigger (Disparador)
- **Push a main**: Cada vez que hagas push a la rama `main`
- **Manual**: Puedes ejecutarlo manualmente desde GitHub Actions

### Pasos del Workflow
1. **📦 Checkout**: Descarga el código
2. **🏗️ Setup Node.js**: Configura Node.js 18
3. **📋 Install**: Instala dependencias con `npm ci`
4. **🧪 Tests**: Ejecuta tests (opcional)
5. **📏 Lint**: Verifica calidad de código
6. **🏗️ Build**: Construye la aplicación
7. **🚀 Deploy**: Se conecta al VPS y despliega

### Proceso en el VPS
1. Pull del código más reciente
2. Instalación de dependencias
3. Build de la aplicación
4. Parada de contenedores Docker
5. Limpieza de imágenes viejas
6. Construcción y arranque de nuevos contenedores
7. Verificación de que todo funciona
8. Recarga de Nginx

## 📊 Monitoreo y Verificación

### Ver Estado del Deployment

1. Ve a la pestaña **Actions** en tu repositorio de GitHub
2. Verás el estado de cada deployment
3. Haz clic en cualquier workflow para ver los detalles

### Comandos de Verificación en el VPS

```bash
# Verificar estado de contenedores
docker ps

# Ver logs de la aplicación
docker compose -f docker-compose.production.yml logs -f

# Verificar que la aplicación responde
curl -I https://pitutito.cl
curl -I https://studio.pitutito.cl

# Ver logs de Nginx
sudo tail -f /var/log/nginx/access.log
```

## 🚨 Solución de Problemas

### Problema: "Host key verification failed"

```bash
# En el VPS, agregar github.com a known_hosts
ssh-keyscan github.com >> ~/.ssh/known_hosts
```

### Problema: "Permission denied (docker)"

```bash
# En el VPS, verificar permisos de Docker
sudo usermod -aG docker gabriel
newgrp docker
```

### Problema: "Git pull failed"

```bash
# En el VPS, verificar configuración de git
cd /home/gabriel/apps/pitutito
git status
git remote -v

# Si hay conflictos, hacer reset (¡cuidado con cambios locales!)
git reset --hard origin/main
```

### Problema: Workflow falla en el paso de deployment

1. Revisa los logs en GitHub Actions
2. Verifica que los secrets estén correctamente configurados
3. Prueba la conexión SSH manualmente
4. Verifica que el VPS esté funcionando

## 🔒 Consideraciones de Seguridad

### Claves SSH
- ✅ Usa claves SSH específicas para GitHub Actions
- ✅ No compartas las claves privadas
- ✅ Rota las claves periódicamente

### Secrets
- ✅ Solo añade secrets necesarios
- ✅ Usa nombres descriptivos
- ✅ Revisa regularmente quién tiene acceso

### VPS
- ✅ Mantén el VPS actualizado
- ✅ Usa firewall (UFW) configurado correctamente
- ✅ Monitorea logs de acceso

## 🎯 Testing del Setup

### Prueba Completa
1. Haz un cambio pequeño en tu código (ej: cambiar un texto)
2. Haz commit y push a la rama `main`
3. Ve a GitHub Actions y observa el deployment
4. Verifica que el cambio aparezca en https://pitutito.cl

### Comando de Test Manual
```bash
# Desde tu máquina local, probar conexión SSH
ssh -i ~/.ssh/pitutito_deploy gabriel@167.172.251.27 "cd /home/gabriel/apps/pitutito && git status"
```

## 📈 Próximos Pasos

Una vez que el despliegue automatizado funcione:

1. **🔄 Deploy Previews**: Configurar despliegues de preview para pull requests
2. **🧪 Tests Automatizados**: Añadir suite de tests antes del deploy
3. **📊 Monitoring**: Configurar alertas de uptime
4. **🚀 Rollback**: Implementar sistema de rollback rápido
5. **🔄 Blue-Green**: Considerar despliegue blue-green para cero downtime

## 🆘 Contacto y Soporte

Si tienes problemas con la configuración:

1. Revisa los logs de GitHub Actions
2. Verifica la conectividad SSH
3. Contacta: gabriel@pantoja.cl

---

**✅ Una vez configurado correctamente, cada push a `main` desplegará automáticamente a https://pitutito.cl** 🎉