# Acceso al Servidor

## Información de Conexión

**IPv4:** `167.172.251.27`  
**Usuario:** `gabriel`

## Comando de Conexión SSH

```bash
ssh gabriel@167.172.251.27
```

## Usuarios del Sistema

- **gabriel**: Usuario principal con privilegios sudo
- **supabase**: Usuario dedicado para servicios de base de datos y aplicaciones

## Cambio de Usuario

Para cambiar al usuario supabase:
```bash
sudo su - supabase
```

## Notas de Seguridad

- Usar claves SSH para autenticación (recomendado)
- Evitar conexiones con contraseña cuando sea posible
- El usuario supabase tiene acceso a Docker y servicios de aplicación