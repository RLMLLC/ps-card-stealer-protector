# 🛡️ PrestaShop Antimalware Protection Script

**Versión:** 1.0  
**Fecha:** 6 de febrero de 2026  
**Autor:** Desarrollado para protección automática contra Credit Card Stealer

---

## 🌍 Language / Idioma

<div align="center">

[![Volver](https://img.shields.io/badge/⬅️_Volver_al_inicio-gray?style=for-the-badge)](README_ANTIMALWARE.md)
&nbsp;&nbsp;
[![English](https://img.shields.io/badge/🇬🇧_English-blue?style=for-the-badge)](README_EN.md)

</div>

---

## 📋 ÍNDICE

1. [Descripción](#descripción)
2. [Características](#características)
3. [Requisitos](#requisitos)
4. [Instalación](#instalación)
5. [Configuración](#configuración)
6. [Uso](#uso)
7. [Configuración de Cron](#configuración-de-cron)
8. [Logs y Monitoreo](#logs-y-monitoreo)
9. [Troubleshooting](#troubleshooting)
10. [Seguridad](#seguridad)
11. [Preguntas Frecuentes](#preguntas-frecuentes)

---

## 🎯 DESCRIPCIÓN

Este script protege automáticamente tu instalación de PrestaShop contra el malware tipo **"Credit Card Stealer"** que:

- Infecta archivos PHP core (Controller.php, FrontController.php)
- Roba datos de tarjetas de crédito durante el checkout
- Se disfraza como imágenes falsas en el directorio `/img/`
- Envía datos robados a servidores en China

**El script:**
- ✅ Detecta infecciones automáticamente
- ✅ Restaura archivos infectados desde URLs seguras
- ✅ Elimina malware disfrazado de imágenes
- ✅ Crea backups antes de modificar archivos
- ✅ Notifica por email cuando detecta problemas
- ✅ Se ejecuta automáticamente vía cron

---

## ⚙️ CARACTERÍSTICAS

### Detección de Malware PHP
- Busca patrones específicos en archivos críticos:
  - `jschecks`, `order_llx`, IPs codificadas en Base64
  - Referencias a servidores C&C (106.14.40.200, 47.102.208.65)
  - Código ofuscado característico del stealer

### Restauración Automática
- Descarga archivos limpios desde URLs configurables
- Verifica que los archivos descargados sean PHP válidos
- Crea backups automáticos antes de cualquier modificación
- Preserva permisos correctos (644)

### Detección de Imágenes Falsas
Detecta malware disfrazado mediante **4 métodos**:
1. **Análisis de tipo de archivo:** Detecta archivos ASCII marcados como PNG/JPG
2. **Búsqueda de patrones:** Identifica código JavaScript/PHP en imágenes
3. **Validación de headers PNG:** Verifica headers mágicos (89504e47)
4. **Validación de headers JPEG:** Verifica headers mágicos (ffd8)

### Sistema de Logs
- Log detallado de todas las operaciones
- Rotación automática al llegar a 10MB
- Niveles de log: INFO, ALERT, SUCCESS, ERROR, WARNING

### Notificaciones
- Email automático cuando detecta infección
- Opcional: email cuando todo está limpio
- Compatible con `mail` y `sendmail`

---

## 📦 REQUISITOS

### Software necesario:
- ✅ Bash shell (Linux/Unix)
- ✅ curl (para descargar archivos)
- ✅ grep, sed, awk (utilidades estándar)
- ✅ Acceso SSH al servidor
- ✅ Permisos de escritura en PrestaShop
- ⚠️ OPCIONAL: mail/sendmail (para notificaciones)

### Verificar requisitos:
```bash
# Verificar que curl está disponible
which curl

# Verificar bash
which bash

# Verificar permisos de escritura
touch /home/usuario/test.txt && rm /home/usuario/test.txt
```

---

## 🚀 INSTALACIÓN

### Paso 1: Preparar archivos limpios de PrestaShop

**IMPORTANTE:** Los archivos PHP no pueden servirse directamente como descarga porque el servidor los ejecuta como código. Debes renombrarlos a `.txt`.

1. **Descarga tu versión exacta de PrestaShop:**
   ```bash
   # Desde GitHub (ejemplo para 1.7.8.7)
   wget https://github.com/PrestaShop/PrestaShop/archive/refs/tags/1.7.8.7.zip
   unzip 1.7.8.7.zip
   cd PrestaShop-1.7.8.7
   ```

2. **Extrae los 3 archivos necesarios:**
   ```bash
   mkdir ~/prestashop_clean
   cp classes/controller/Controller.php ~/prestashop_clean/
   cp classes/controller/FrontController.php ~/prestashop_clean/
   cp controllers/admin/AdminLoginController.php ~/prestashop_clean/
   ```

3. **Renombra a .txt para servir como texto plano:**
   ```bash
   cd ~/prestashop_clean
   mv Controller.php Controller.php.txt
   mv FrontController.php FrontController.php.txt
   mv AdminLoginController.php AdminLoginController.php.txt
   ```

4. **Sube a una URL accesible:**
   
   **Opción A - Subdirectorio en tu hosting:**
   ```bash
   # Crear directorio protegido
   mkdir -p /home/usuario/public_html/clean_files
   
   # Subir archivos
   cp ~/prestashop_clean/*.txt /home/usuario/public_html/clean_files/
   
   # URLs finales:
   # https://tudominio.com/clean_files/Controller.php.txt
   # https://tudominio.com/clean_files/FrontController.php.txt
   # https://tudominio.com/clean_files/AdminLoginController.php.txt
   ```

   **Opción B - Hosting externo seguro (RECOMENDADO):**
   - Sube a un servidor separado
   - Usa un subdominio: `https://clean.tudominio.com/`
   - Protege con .htaccess si es necesario

### Paso 2: Subir el script al servidor

```bash
# Via SCP
scp prestashop_antimalware.sh usuario@tuservidor.com:/home/usuario/security/

# Via FTP/SFTP
# Usa tu cliente FTP preferido (FileZilla, WinSCP, etc.)
```

### Paso 3: Dar permisos de ejecución

```bash
chmod +x /home/usuario/security/prestashop_antimalware.sh
```

### Paso 4: Crear directorios necesarios

```bash
# Directorio de logs
mkdir -p /home/usuario/logs

# Directorio de backups
mkdir -p /home/usuario/var/backups/infected

# Directorio temporal
mkdir -p /home/usuario/public_html/var/files

# Verificar permisos
chmod 755 /home/usuario/logs
chmod 755 /home/usuario/var/backups/infected
chmod 755 /home/usuario/public_html/var/files
```

---

## ⚙️ CONFIGURACIÓN

Edita el script y modifica las variables en las **líneas 14-39**:

```bash
nano /home/usuario/security/prestashop_antimalware.sh
```

### Variables a configurar:

```bash
# === CONFIGURACIÓN - EDITA ESTOS VALORES ===

# 1. Ruta a tu instalación de PrestaShop
PRESTASHOP_ROOT="/home/usuario/public_html"

# 2. URLs de archivos limpios (IMPORTANTE: deben terminar en .txt)
CLEAN_CONTROLLER_URL="https://tudominio.com/clean_files/Controller.php.txt"
CLEAN_FRONTCONTROLLER_URL="https://tudominio.com/clean_files/FrontController.php.txt"
CLEAN_ADMINLOGIN_URL="https://tudominio.com/clean_files/AdminLoginController.php.txt"

# 3. Email para notificaciones
EMAIL_ALERTS="admin@tudominio.com"

# 4. ¿Enviar email cuando detecta infección?
SEND_EMAIL_ON_INFECTION="yes"

# 5. ¿Enviar email cuando todo está limpio?
SEND_EMAIL_ON_CLEAN="no"  # Recomendado "no" si usas cron cada minuto

# 6. Ubicación del log
LOG_FILE="/home/usuario/logs/prestashop_antimalware.log"
LOG_MAX_SIZE=10485760  # 10MB

# 7. Directorio para backups
BACKUP_DIR="/home/usuario/var/backups/infected"

# 8. Directorio temporal para descargas
TEMP_DIR="/home/usuario/public_html/var/files"

# 9. Verificar certificados SSL
VERIFY_SSL="yes"  # Cambiar a "no" si tienes problemas con SSL
```

### Ejemplo de configuración completa:

```bash
PRESTASHOP_ROOT="/home/sc1gijo7672/public_html"
CLEAN_CONTROLLER_URL="https://rlm.llc/ps/1787/Controller.php.txt"
CLEAN_FRONTCONTROLLER_URL="https://rlm.llc/ps/1787/FrontController.php.txt"
CLEAN_ADMINLOGIN_URL="https://rlm.llc/ps/1787/AdminLoginController.php.txt"
EMAIL_ALERTS="admin@mitienda.com"
SEND_EMAIL_ON_INFECTION="yes"
SEND_EMAIL_ON_CLEAN="no"
LOG_FILE="/home/sc1gijo7672/logs/prestashop_antimalware.log"
BACKUP_DIR="/home/sc1gijo7672/var/backups/infected"
TEMP_DIR="/home/sc1gijo7672/public_html/var/files"
VERIFY_SSL="yes"
```

---

## 🎮 USO

### Ejecución Manual

```bash
# Ejecutar el script manualmente
bash /home/usuario/security/prestashop_antimalware.sh
```

**Salida esperada si todo está limpio:**
```
════════════════════════════════════════════════════════════
  PrestaShop Antimalware Protection - Escaneo Iniciado
  2026-02-06 18:05:53
════════════════════════════════════════════════════════════

[1/4] Verificando Controller.php...
[2/4] Verificando FrontController.php...
[3/4] Verificando AdminLoginController.php...
[4/4] Escaneando directorio /img/...
ℹ Escaneando directorio /img/ (no recursivo)...
✓ Directorio /img/ limpio

════════════════════════════════════════════════════════════
  RESUMEN DEL ESCANEO
════════════════════════════════════════════════════════════
✓ SISTEMA LIMPIO
  No se detectó malware
════════════════════════════════════════════════════════════
```

**Salida si detecta infección:**
```
════════════════════════════════════════════════════════════
  PrestaShop Antimalware Protection - Escaneo Iniciado
  2026-02-06 18:05:53
════════════════════════════════════════════════════════════

[1/4] Verificando Controller.php...
⚠ INFECCIÓN DETECTADA: Controller.php
↻ Descargando archivo limpio: Controller.php
✓ Backup guardado: /home/.../Controller.php.infected.20260206_180554
✓ Restaurado exitosamente: Controller.php

[2/4] Verificando FrontController.php...
⚠ INFECCIÓN DETECTADA: FrontController.php
↻ Descargando archivo limpio: FrontController.php
✓ Backup guardado: /home/.../FrontController.php.infected.20260206_180554
✓ Restaurado exitosamente: FrontController.php

[3/4] Verificando AdminLoginController.php...
[4/4] Escaneando directorio /img/...
ℹ Escaneando directorio /img/ (no recursivo)...
⚠ MALWARE DETECTADO: fake_logo.jpg
✓ Eliminado: fake_logo.jpg

════════════════════════════════════════════════════════════
  RESUMEN DEL ESCANEO
════════════════════════════════════════════════════════════
⚠ INFECCIÓN DETECTADA Y LIMPIADA
  Archivos PHP infectados: 2
  Archivos PHP restaurados: 2
  Archivos malware eliminados: 1

  Los archivos infectados han sido respaldados en:
  /home/usuario/var/backups/infected
════════════════════════════════════════════════════════════
```

---

## ⏰ CONFIGURACIÓN DE CRON

Para protección automática 24/7, configura un cron job.

### Opción A: Via cPanel

1. Acceder a **cPanel → Cron Jobs**
2. Configurar frecuencia:
   - **Cada minuto (RECOMENDADO):**
     - Minuto: `*`
     - Hora: `*`
     - Día: `*`
     - Mes: `*`
     - Día semana: `*`
   
3. **Comando:**
   ```bash
   /bin/bash /home/usuario/security/prestashop_antimalware.sh >> /home/usuario/logs/cron_output.log 2>&1
   ```

4. **Guardar**

### Opción B: Via SSH (crontab)

```bash
# Editar crontab
crontab -e

# Añadir una de estas líneas:

# Cada minuto (RECOMENDADO para máxima protección)
* * * * * /bin/bash /home/usuario/security/prestashop_antimalware.sh >> /home/usuario/logs/cron_output.log 2>&1

# Cada 5 minutos
*/5 * * * * /bin/bash /home/usuario/security/prestashop_antimalware.sh >> /home/usuario/logs/cron_output.log 2>&1

# Cada 15 minutos
*/15 * * * * /bin/bash /home/usuario/security/prestashop_antimalware.sh >> /home/usuario/logs/cron_output.log 2>&1

# Solo en horario comercial (9am-6pm, lunes a viernes)
* 9-18 * * 1-5 /bin/bash /home/usuario/security/prestashop_antimalware.sh >> /home/usuario/logs/cron_output.log 2>&1
```

### Verificar que el cron está activo:

```bash
# Listar crons activos
crontab -l

# Esperar 1-2 minutos y verificar log
tail -f /home/usuario/logs/cron_output.log
```

---

## 📊 LOGS Y MONITOREO

### Ver logs en tiempo real:

```bash
# Log principal del script
tail -f /home/usuario/logs/prestashop_antimalware.log

# Output del cron
tail -f /home/usuario/logs/cron_output.log
```

### Buscar infecciones detectadas:

```bash
# Ver todas las alertas
grep "ALERT" /home/usuario/logs/prestashop_antimalware.log

# Ver estadísticas
grep "STATS" /home/usuario/logs/prestashop_antimalware.log

# Ver últimas 50 líneas
tail -50 /home/usuario/logs/prestashop_antimalware.log
```

### Ver backups creados:

```bash
# Listar backups
ls -lah /home/usuario/var/backups/infected/

# Ver contenido de un backup
less /home/usuario/var/backups/infected/Controller.php.infected.20260206_180554
```

### Limpiar logs antiguos:

```bash
# Ver tamaño del log
du -h /home/usuario/logs/prestashop_antimalware.log

# Rotar manualmente si es muy grande
mv /home/usuario/logs/prestashop_antimalware.log \
   /home/usuario/logs/prestashop_antimalware.log.old
touch /home/usuario/logs/prestashop_antimalware.log
```

---

## 🔧 TROUBLESHOOTING

### Problema 1: "Permission denied"

**Síntoma:**
```
bash: /home/usuario/security/prestashop_antimalware.sh: Permission denied
```

**Solución:**
```bash
chmod +x /home/usuario/security/prestashop_antimalware.sh
```

---

### Problema 2: "Error al descargar archivo limpio"

**Síntoma:**
```
✗ Error al descargar archivo limpio
```

**Causa:** Los archivos PHP se están ejecutando en vez de descargarse.

**Solución:**
1. **Asegúrate de que las URLs terminan en .txt:**
   ```bash
   CLEAN_CONTROLLER_URL="https://tudominio.com/files/Controller.php.txt"
   # NO: Controller.php
   ```

2. **Verifica que los archivos están renombrados en el servidor:**
   ```bash
   # En el servidor donde alojaste los archivos
   ls -la /path/to/clean_files/
   # Deberías ver: Controller.php.txt, FrontController.php.txt, etc.
   ```

3. **Test manual:**
   ```bash
   curl https://tudominio.com/files/Controller.php.txt | head -20
   # Debería mostrar código PHP, NO ejecutarlo
   ```

---

### Problema 3: "No such file or directory" en directorio temporal

**Síntoma:**
```
head: impossible d'ouvrir '/home/usuario/public_html/var/files/ps_clean_123.php'
```

**Causa:** El directorio temporal no existe o no tiene permisos.

**Solución:**
```bash
# Crear directorio
mkdir -p /home/usuario/public_html/var/files
chmod 755 /home/usuario/public_html/var/files

# Verificar que TEMP_DIR está bien configurado en el script
grep TEMP_DIR /home/usuario/security/prestashop_antimalware.sh
```

---

### Problema 4: "curl: command not found"

**Síntoma:**
```
bash: curl: command not found
```

**Solución:**
```bash
# Ubuntu/Debian
sudo apt-get install curl

# CentOS/RedHat
sudo yum install curl

# Si no tienes acceso root, contacta a tu proveedor de hosting
```

---

### Problema 5: No recibo emails

**Síntoma:** El script funciona pero no llegan notificaciones.

**Solución:**

1. **Verificar que mail está instalado:**
   ```bash
   which mail
   # Si no devuelve nada:
   sudo apt-get install mailutils
   ```

2. **Cambiar configuración:**
   ```bash
   # Si no necesitas emails, desactívalos:
   SEND_EMAIL_ON_INFECTION="no"
   ```

3. **Test manual:**
   ```bash
   echo "Test email" | mail -s "Test" tu@email.com
   ```

---

### Problema 6: "SSL certificate problem"

**Síntoma:**
```
curl: (60) SSL certificate problem: unable to get local issuer certificate
```

**Solución:**
```bash
# En el script, cambiar:
VERIFY_SSL="no"
```

---

### Problema 7: Archivos temporales se acumulan

**Síntoma:** El directorio `var/files/` tiene muchos archivos `ps_clean_*.php`.

**Causa:** Si curl falla, los archivos temporales no se limpian.

**Solución:**
```bash
# Limpiar archivos temporales manualmente
rm -f /home/usuario/public_html/var/files/ps_clean_*.php

# O añadir al cron (se limpia automáticamente cada día):
0 0 * * * find /home/usuario/public_html/var/files/ps_clean_*.php -mtime +1 -delete 2>/dev/null
```

---

## 🔒 SEGURIDAD

### Protección de archivos limpios

Si alojas los archivos limpios en tu propio servidor, protégelos:

```bash
# Crear .htaccess en el directorio
cat > /home/usuario/public_html/clean_files/.htaccess << 'EOF'
# Permitir acceso solo desde tu servidor
Order Deny,Allow
Deny from all
Allow from 123.456.789.012
# Reemplaza con la IP de tu servidor PrestaShop
EOF
```

### Permisos recomendados

```bash
# Script de protección
chmod 500 /home/usuario/security/prestashop_antimalware.sh

# Directorio de backups
chmod 700 /home/usuario/var/backups/infected

# Logs
chmod 644 /home/usuario/logs/prestashop_antimalware.log
```

### Backups de seguridad

```bash
# Los backups se guardan automáticamente con timestamp:
# Controller.php.infected.20260206_180554
# FrontController.php.infected.20260206_180554
# fake_logo.jpg.malware.20260206_180555

# IMPORTANTE: Los backups contienen malware, no los ejecutes
# Son solo para análisis forense o recuperación si hay falso positivo
```

---

## ❓ PREGUNTAS FRECUENTES

### ¿Por qué necesito renombrar los archivos PHP a .txt?

Los servidores web ejecutan archivos `.php` como código. Si intentas descargar `Controller.php`, el servidor lo ejecutará y devolverá su output (usualmente vacío o un error), no el código fuente. Renombrándolo a `.txt`, el servidor lo sirve como texto plano.

### ¿Es seguro guardar los backups de archivos infectados?

Sí, son solo archivos de texto y no se ejecutan automáticamente. Son útiles para:
- Análisis forense
- Evidencia legal
- Recuperación si hay un falso positivo

Puedes eliminarlos manualmente cuando quieras:
```bash
rm -rf /home/usuario/var/backups/infected/*
```

### ¿Cada cuánto debe ejecutarse el script?

**Recomendación:** Cada 1 minuto para máxima protección.

El script es muy ligero (< 1 segundo de ejecución) y solo envía email cuando detecta problemas.

### ¿Qué pasa si el malware vuelve a infectar?

El script detectará y limpiará la infección automáticamente en el próximo ciclo (máximo 1 minuto si usas cron cada minuto).

**Sin embargo**, debes investigar:
- ¿Cómo entró el atacante?
- ¿Tiene acceso FTP/SSH comprometido?
- ¿Módulos vulnerables instalados?
- ¿Contraseñas débiles?

### ¿Protege contra todas las infecciones?

NO. Este script protege específicamente contra el malware "Credit Card Stealer" analizado. Para protección completa:
- Actualiza PrestaShop a la última versión
- Usa contraseñas fuertes
- Mantén módulos actualizados
- Implementa WAF (Cloudflare)
- Haz backups regulares

### ¿Puedo modificar el script?

Sí, el script es completamente modificable. Documentación de funciones principales:

- `detect_malware_in_php()` - Detecta patrones maliciosos
- `restore_file_from_url()` - Descarga y restaura archivos
- `is_fake_image()` - Detecta imágenes falsas
- `scan_img_directory()` - Escanea /img/

### ¿Funciona con cualquier versión de PrestaShop?

El script funciona con cualquier versión, PERO necesitas usar archivos limpios de tu versión exacta de PrestaShop.

### ¿Qué hacer después de la primera limpieza?

1. ✅ Cambiar TODAS las contraseñas (FTP, SSH, DB, Admin)
2. ✅ Revisar usuarios con acceso al servidor
3. ✅ Auditar módulos instalados
4. ✅ Actualizar PrestaShop si es posible
5. ✅ Implementar WAF (Cloudflare gratis)
6. ✅ Configurar backups automáticos
7. ✅ Monitorear logs durante 2 semanas

---

## 📞 SOPORTE

### Verificar estado del sistema

```bash
# Ejecutar manualmente
bash /home/usuario/security/prestashop_antimalware.sh

# Ver últimas 100 líneas del log
tail -100 /home/usuario/logs/prestashop_antimalware.log

# Buscar errores
grep "ERROR" /home/usuario/logs/prestashop_antimalware.log

# Verificar que el cron funciona
crontab -l
```

### Archivos importantes

- **Script:** `/home/usuario/security/prestashop_antimalware.sh`
- **Log principal:** `/home/usuario/logs/prestashop_antimalware.log`
- **Log del cron:** `/home/usuario/logs/cron_output.log`
- **Backups:** `/home/usuario/var/backups/infected/`
- **Archivos temporales:** `/home/usuario/public_html/var/files/`

---

## 📝 CHANGELOG

### Versión 1.0 (2026-02-06)
- ✅ Detección de malware en 3 archivos PHP críticos
- ✅ Restauración automática desde URLs configurables
- ✅ Detección de imágenes falsas con 4 métodos
- ✅ Sistema de backups automático
- ✅ Logs detallados con rotación
- ✅ Notificaciones por email
- ✅ Variables configurables (incluyendo TEMP_DIR)
- ✅ Creación automática de directorios necesarios
- ✅ Validación mejorada de archivos PHP (primeras 20 líneas)
- ✅ Soporte para archivos .txt como fuente limpia

---

## 📜 LICENCIA

Este script se proporciona "tal cual" sin garantías de ningún tipo.
Úsalo bajo tu propia responsabilidad.

---

## ✅ CHECKLIST DE INSTALACIÓN

- [ ] Archivos limpios de PrestaShop descargados
- [ ] Archivos renombrados a .txt
- [ ] Archivos subidos a URL accesible
- [ ] Script subido al servidor
- [ ] Permisos de ejecución configurados (chmod +x)
- [ ] Directorios creados (logs, backups, temp)
- [ ] Script configurado (todas las variables editadas)
- [ ] Test manual ejecutado con éxito
- [ ] Cron job configurado
- [ ] Email de prueba recibido (si aplica)
- [ ] Logs verificados durante 24 horas
- [ ] Contraseñas cambiadas
- [ ] Plan de actualización de PrestaShop preparado

---

<div align="center">

**¡Tu PrestaShop está ahora protegido contra reinfecciones automáticas!** 🛡️

[![Volver al inicio](https://img.shields.io/badge/⬅️_Volver_al_inicio-gray?style=for-the-badge)](README_ANTIMALWARE.md)
&nbsp;&nbsp;
[![English Version](https://img.shields.io/badge/🇬🇧_Read_in_English-blue?style=for-the-badge)](README_EN.md)

</div>
