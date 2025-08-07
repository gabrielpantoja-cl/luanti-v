# 🌐 Instrucciones de Configuración DNS para pitutito.cl

## 📋 Registros DNS Requeridos

### Configuración Básica para VPS
Necesitas configurar estos registros DNS en tu proveedor de dominio donde registraste `pitutito.cl`:

| Tipo  | Nombre | Valor           | TTL  | Descripción                    |
|-------|--------|-----------------|------|--------------------------------|
| A     | @      | 167.172.251.27  | 3600 | Dominio principal (pitutito.cl)|
| A     | www    | 167.172.251.27  | 3600 | Subdominio www                 |
| CNAME | api    | pitutito.cl     | 3600 | API de Supabase                |
| CNAME | studio | pitutito.cl     | 3600 | Dashboard de Supabase          |

### Configuración Avanzada (Opcional)
Para mayor flexibilidad, también puedes agregar:

| Tipo  | Nombre | Valor           | TTL  | Descripción                    |
|-------|--------|-----------------|------|--------------------------------|
| A     | *      | 167.172.251.27  | 3600 | Wildcard para subdominios      |
| TXT   | @      | "v=spf1 -all"   | 3600 | Seguridad anti-spam            |

## 🏢 Proveedores de DNS Comunes

### NIC Chile (.cl)
1. Ingresa a tu panel de control de NIC Chile
2. Ve a "Gestión de DNS" o "Zona DNS"
3. Agrega los registros según la tabla anterior
4. Guarda los cambios
5. Espera propagación (5-60 minutos para .cl)

### Cloudflare (Recomendado)
1. Ve a tu dashboard de Cloudflare: https://dash.cloudflare.com
2. Selecciona el dominio `pitutito.cl`
3. Ve a la sección "DNS" > "Records"
4. Agrega los registros con estos detalles específicos:

```
┌────────────────────────────────────────────────────┐
│  TIPO    │  NOMBRE   │  CONTENIDO      │  PROXY  │
├────────────────────────────────────────────────────┤
│  A       │  @        │  167.172.251.27 │   🟠   │
│  A       │  www      │  167.172.251.27 │   🟠   │
│  A       │  api      │  167.172.251.27 │   🟠   │
│  A       │  studio   │  167.172.251.27 │   🟠   │
└────────────────────────────────────────────────────┘
```

⚠️ **IMPORTANTE**: En Cloudflare, activa el PROXY (nube naranja 🟠) para obtener:
- SSL/HTTPS automático
- Protección DDoS
- CDN global
- Analytics avanzados

### GoDaddy
1. Accede a tu cuenta de GoDaddy
2. Ve a "Mis productos" > "DNS"
3. Selecciona tu dominio `pitutito.cl`
4. Agrega los registros DNS
5. Tiempo de propagación: 1-48 horas

### Otros Proveedores
El proceso es similar en la mayoría de proveedores:
- Accede al panel de control
- Busca "DNS Management", "Zone File", o "DNS Records"
- Agrega los registros según la tabla
- Guarda y espera propagación

## ⏰ Tiempos de Propagación

| Proveedor    | Propagación Local | Propagación Global |
|--------------|-------------------|-------------------|
| Cloudflare   | 1-5 minutos       | 5-30 minutos      |
| NIC Chile    | 5-60 minutos      | 1-24 horas        |
| GoDaddy      | 1-2 horas         | 1-48 horas        |
| Genérico     | 30 minutos        | 24-48 horas       |

## 🔍 Verificación de Propagación

### Herramientas Online
- **DNS Checker**: https://dnschecker.org
- **What's My DNS**: https://www.whatsmydns.net
- **DNS Propagation**: https://dnspropagation.net

### Comandos Terminal
```bash
# Verificar registro A principal
nslookup pitutito.cl

# Verificar subdominios
nslookup www.pitutito.cl
nslookup api.pitutito.cl
nslookup studio.pitutito.cl

# Verificar con dig (más detallado)
dig pitutito.cl
dig +trace pitutito.cl
```

### Resultados Esperados
```bash
# Comando: nslookup pitutito.cl
Name:    pitutito.cl
Address: 167.172.251.27

# Comando: nslookup api.pitutito.cl  
Name:    api.pitutito.cl
Address: 167.172.251.27
```

## 🚨 Troubleshooting

### DNS no propaga
1. **Verificar sintaxis**: Asegúrate de que no haya espacios extra o caracteres especiales
2. **TTL alto**: Si cambias registros existentes, el TTL anterior puede causar demora
3. **Cache local**: Limpia el cache DNS local:
   ```bash
   # Windows
   ipconfig /flushdns
   
   # macOS
   sudo dscacheutil -flushcache
   
   # Linux
   sudo systemctl restart systemd-resolved
   ```

### Error "Domain not found"
1. Verifica que el dominio esté activo en tu proveedor
2. Asegúrate de que los nameservers apunten al proveedor correcto
3. Contacta soporte de tu proveedor de dominio

### Subdominios no resuelven
1. Verifica que los registros CNAME o A estén correctos
2. Algunos proveedores requieren el punto final: `pitutito.cl.`
3. Verifica que no haya conflictos con registros existentes

## 🎯 Después de Configurar DNS

### 1. Esperar Propagación
- Mínimo 5 minutos (Cloudflare)
- Máximo 48 horas (otros proveedores)

### 2. Verificar Resolución
```bash
nslookup pitutito.cl
nslookup api.pitutito.cl
```

### 3. Ejecutar Deployment
Una vez que el DNS funcione, ejecutar:
```bash
./deploy-with-domain.sh
```

### 4. URLs Finales
- 🏠 **https://pitutito.cl** - Aplicación principal
- 🌍 **https://www.pitutito.cl** - Redirect a principal
- 🔌 **https://api.pitutito.cl** - API de Supabase
- 📊 **https://studio.pitutito.cl** - Dashboard

## 📞 Soporte por Proveedor

### Cloudflare
- **Docs**: https://developers.cloudflare.com/dns/
- **Support**: https://support.cloudflare.com/

### NIC Chile
- **Portal**: https://www.nic.cl
- **Email**: info@nic.cl
- **Teléfono**: +56 2 2940 7700

### GoDaddy
- **Help**: https://www.godaddy.com/help
- **Chat**: Disponible 24/7

---

**💡 Tip**: Si tienes el dominio en NIC Chile pero quieres usar Cloudflare, puedes cambiar los nameservers a Cloudflare y gestionar todo el DNS desde allí para mejor performance y seguridad.

**Próximo paso**: Una vez configurado el DNS, ejecutar `./deploy-with-domain.sh` para completar el deployment con dominio personalizado.