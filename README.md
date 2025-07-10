# 🚀 Odoo 18 Deployment en Google Cloud Platform

[![Deploy to GCP](https://img.shields.io/badge/Deploy%20to%20GCP-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](../../actions/workflows/deploy.yml)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)
[![Odoo Version](https://img.shields.io/badge/Odoo-18.0-875A7B.svg?style=for-the-badge&logo=odoo)](https://www.odoo.com)

## 📋 Descripción

Este repositorio contiene un workflow de GitHub Actions que automatiza el despliegue de **Odoo 18** en Google Cloud Platform (GCP) con una sola acción.

## ✨ Características

- 🔧 **Instalación automática** de Odoo 18 Community
- 🐘 **PostgreSQL** preconfigurado
- 🌐 **Nginx** como proxy reverso
- 🔒 **Firewall** configurado automáticamente
- 📊 **Monitoreo** y logging incluido
- 🏷️ **Nombres únicos** con timestamp
- 💾 **Configuración persistente**

## 🚀 Despliegue Rápido

### Opción 1: Usar el botón de despliegue

1. Haz clic en el botón **"Deploy to GCP"** arriba
2. Selecciona los parámetros deseados:
   - **Nombre de instancia**: Nombre base (se añadirá timestamp)
   - **Tipo de máquina**: e2-micro, e2-small, e2-medium, etc.
   - **Zona**: us-central1-a o southamerica-west1-a
   - **Tamaño de disco**: En GB (mínimo 10GB)
3. Haz clic en **"Run workflow"**
4. Espera 10-15 minutos para que la instalación complete

### Opción 2: Ejecutar manualmente

```bash
# Navegar a Actions > Deploy Odoo 18 to Google Cloud > Run workflow
```

## 🛠️ Configuración Inicial

### Prerrequisitos

1. **Cuenta de Google Cloud Platform** con proyecto activo
2. **Service Account** con permisos necesarios
3. **Secrets configurados** en el repositorio de GitHub

### Secrets Requeridos

Configura estos secrets en tu repositorio de GitHub (`Settings > Secrets and variables > Actions`):

| Secret | Descripción | Ejemplo |
|--------|-------------|---------|
| `GCP_PROJECT_ID` | ID del proyecto de GCP | `mi-proyecto-123456` |
| `GCP_SA_KEY` | Clave JSON del Service Account | `{"type": "service_account",...}` |
| `GCP_SERVICE_ACCOUNT_EMAIL` | Email del Service Account | `deploy@mi-proyecto.iam.gserviceaccount.com` |

### Permisos del Service Account

El Service Account debe tener estos roles:

- `Compute Admin` (roles/compute.admin)
- `Security Admin` (roles/iam.securityAdmin)
- `Service Account User` (roles/iam.serviceAccountUser)

## 📊 Tipos de Instancia Disponibles

| Tipo | vCPUs | RAM | Uso Recomendado |
|------|-------|-----|-----------------|
| e2-micro | 0.25-2 | 1 GB | Pruebas/Demo |
| e2-small | 0.5-2 | 2 GB | Desarrollo |
| e2-medium | 1-2 | 4 GB | Producción pequeña |
| e2-standard-2 | 2 | 8 GB | Producción media |
| e2-standard-4 | 4 | 16 GB | Producción alta |

## 🌐 Acceso a Odoo

Una vez completado el despliegue:

1. **Busca la IP externa** en los logs del workflow
2. **Accede via web**: `http://[IP_EXTERNA]:8069`
3. **Credenciales por defecto**:
   - Usuario: `admin`
   - Contraseña: `admin`
   - Base de datos: Crear nueva durante el primer acceso

## 📁 Estructura del Proyecto

```
.
├── .github/
│   └── workflows/
│       └── deploy-odoo-gcp.yml    # Workflow principal
├── startup-script.sh              # Script de instalación
├── README.md                      # Este archivo
└── LICENSE                        # Licencia del proyecto
```

## 🔧 Configuración Avanzada

### Modificar la Configuración de Odoo

El archivo de configuración se encuentra en `/etc/odoo/odoo.conf`:

```ini
[options]
admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = odoo
db_password = False
addons_path = /opt/odoo/addons
logfile = /var/log/odoo/odoo.log
log_level = info
```

### Comandos Útiles

```bash
# Conectar via SSH
gcloud compute ssh [NOMBRE_INSTANCIA] --zone=[ZONA]

# Ver logs de Odoo
sudo tail -f /var/log/odoo/odoo.log

# Reiniciar Odoo
sudo systemctl restart odoo

# Ver estado de servicios
sudo systemctl status odoo postgresql nginx

# Eliminar instancia
gcloud compute instances delete [NOMBRE_INSTANCIA] --zone=[ZONA]
```

## 🛡️ Seguridad

### Firewall Configurado

- **Puerto 22**: SSH (restringido)
- **Puerto 80**: HTTP (Nginx)
- **Puerto 8069**: Odoo (directo)

### Recomendaciones de Seguridad

1. **Cambiar contraseña** de admin inmediatamente
2. **Configurar SSL/TLS** para producción
3. **Restringir acceso** por IP si es necesario
4. **Hacer backups** regulares de la base de datos

## 📈 Monitoreo

### Logs Disponibles

- **Odoo**: `/var/log/odoo/odoo.log`
- **Nginx**: `/var/log/nginx/access.log`
- **Sistema**: `/var/log/syslog`
- **Instalación**: `/var/log/odoo-installation.log`

### Comandos de Monitoreo

```bash
# Uso de recursos
htop

# Espacio en disco
df -h

# Procesos de Odoo
ps aux | grep odoo

# Conexiones activas
netstat -tuln | grep -E ":(80|8069|5432)"
```

## 🆘 Troubleshooting

### Problemas Comunes

| Problema | Solución |
|----------|----------|
| Odoo no responde | `sudo systemctl restart odoo` |
| Error de base de datos | Verificar PostgreSQL: `sudo systemctl status postgresql` |
| Sin acceso web | Verificar firewall y Nginx |
| Instalación incompleta | Revisar logs: `sudo tail -f /var/log/odoo-installation.log` |

### Verificar Instalación

```bash
# Verificar todos los servicios
sudo systemctl status odoo postgresql nginx

# Verificar puertos
sudo netstat -tuln | grep -E ":(80|8069|5432)"

# Verificar logs de instalación
sudo cat /var/log/odoo-installation.log
```

## 🔄 Actualización y Mantenimiento

### Actualizar Odoo

```bash
# Conectar via SSH
cd /opt/odoo
sudo -u odoo git pull origin 18.0
sudo systemctl restart odoo
```

### Backup de Base de Datos

```bash
# Crear backup
sudo -u odoo pg_dump odoo > backup_$(date +%Y%m%d_%H%M%S).sql

# Restaurar backup
sudo -u odoo psql odoo < backup_YYYYMMDD_HHMMSS.sql
```

## 📞 Soporte

Si encuentras problemas:

1. **Revisa los logs** del workflow en GitHub Actions
2. **Verifica la configuración** de secrets
3. **Consulta la documentación** de Odoo
4. **Crea un issue** en este repositorio

## 🤝 Contribuciones

¡Las contribuciones son bienvenidas! Por favor:

1. Fork el repositorio
2. Crea una rama para tu feature
3. Commit tus cambios
4. Push a la rama
5. Abre un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver [LICENSE](LICENSE) para más detalles.

## 🙏 Agradecimientos

- [Odoo Community](https://www.odoo.com) por el excelente ERP
- [Google Cloud Platform](https://cloud.google.com) por la infraestructura
- [GitHub Actions](https://github.com/features/actions) por la automatización

---

**¿Listo para desplegar Odoo en segundos?** 👆 ¡Haz clic en el botón de arriba!

---

### 📊 Estado del Último Despliegue

![Workflow Status](https://github.com/[TU_USUARIO]/[TU_REPO]/actions/workflows/deploy-odoo-gcp.yml/badge.svg)
