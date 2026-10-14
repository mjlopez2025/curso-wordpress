Respuestas: 

# Informe de Relevamiento del Servidor

**Fecha:** 2026-10-14
**Responsable:** Maximiliano López
**Servidor:** SITIOVINCULACION-TEST
**IP:** 172.16.1.62
**Puerto SSH:** 2337
**Entorno:** TEST









## 1. Sistema Operativo

| Componente | Valor |
| Distribución | Debian GNU/Linux 11 (bullseye) |
| Versión | 11 |
| Kernel | 5.10.0-16-amd64 |
| Arquitectura | amd64 |

**Observaciones:**
- Debian 11 está en LTS hasta agosto de 2026.
- Kernel desactualizado 



## 2. Stack LAMP

| Componente | Versión | Estado |
|------------|---------|--------|
| Apache | 2.4.54 | Desactualizado |
| PHP | 7.4.33 |  EOL desde nov 2022 |
| MySQL | 8.0.30 | ⚠ Desactualizado |
| Módulo PHP | libapache2-mod-php7.4 |  Prefork, no PHP-FPM |

**Observaciones:**
- PHP corre como módulo de Apache (`mod_php`), no como PHP-FPM.
- Apache usa `mpm_prefork`, no `mpm_event`.
- PHP 7.4 está EOL, sin parches de seguridad.

---

## 3. Usuarios del Sistema

**Usuarios humanos 

| Usuario | UID | Home | Comentario |
|---------|-----|------|------------|
| netadmin | 1000 | /home/netadmin | Admin de red |
| mgarcia | 1001 | /home/mgarcia | |
| jotegui | 1002 | /home/jotegui | |
| vinculacionusrftp | 1003 | /home/vinculacionusrftp | FTP |
| ggomez | 1004 | /home/ggomez | |
| mjlopez | 1005 | /home/mjlopez | (nuevo) |

**Usuarios de sistema relevantes:**
- `www-data` → Apache/PHP
- `mysql` → MySQL
- `sshd` → SSH

**Observaciones:**
- 6 usuarios humanos.
- No hay usuario "admin" en el sistema.

---

## 4. Servicios y Puertos

| Puerto | Servicio | Proceso | Exposición |
|--------|----------|---------|------------|
| 21 | FTP | vsftpd |  Expuesto a cualquier IP |
| 80 | HTTP | apache2 |  Redirige a HTTPS |
| 443 | HTTPS | apache2 | 
| 2337 | SSH | sshd |  No estándar |
| 3306 | MySQL | mysqld |  Expuesto a cualquier IP |
| 33060 | MySQL X | mysqld |  Expuesto a cualquier IP |

**Observaciones:**
- MySQL escucha en `*:3306` y `*:33060` (debería ser solo `127.0.0.1`).
- FTP en puerto 21 (inseguro, texto plano).

---

## 5. Virtual Hosts de Apache

**Sitios activos (`sites-enabled`):**

| Archivo | Dominio | Tipo |
|---------|---------|------|
| cupundav.conf | cuptest.undav.edu.ar | HTTP → HTTPS |
| cupundav-ssl.conf | cuptest.undav.edu.ar | HTTPS |
| vincutecnologica.conf | vinculaciontecnologicatest.undav.edu.ar | HTTP → HTTPS |
| vincutecnologica-ssl.conf | vinculaciontecnologicatest.undav.edu.ar | HTTPS |
| wptest.conf | wptest.undav.edu.ar | HTTP → HTTPS |
| wptest-ssl.conf | wptest.undav.edu.ar | HTTPS |

**Observaciones:**
- wptest-ssl.conf NO tiene configuración SSL propia.
- Usa el certificado wildcard del servidor.

---

## 6. Sitios en /var/www/

| Directorio | Propósito | Propietario |
|------------|-----------|-------------|
| accesoapache/ | .htpasswd para proteger wp-admin | root |
| bkbases/ | Backups de bases de datos | root |
| html/ | Página default de Apache | root |
| vincutecno/ | cuptest (WordPress) | root:www-data |
| vincutecno2/ | vinculaciontecnologicatest (WordPress) | root:www-data |
| wptest.undav.edu.ar/ | Entorno de práctica | root |

**Observaciones:**
- `bkbases/` contiene backups de bases de datos en `/var/www/` 

---

## 7. Módulos de Apache

**Módulos cargados (30):**

- Base: core, so, http, log_config, logio, version, unixd
- Auth: access_compat, auth_basic, authn_core, authn_file, authz_core, authz_host, authz_user
- Archivos: alias, dir, mime, negotiation
- **autoindex** (listado de directorios)
- **deflate** (compresión)
- env, filter, setenvif, headers
- **mpm_prefork** ( modelo antiguo)
- **php7** (módulo de Apache)
- reqtimeout, rewrite, socache_shmcb, ssl
- **status** (info del servidor)

**Observaciones:**
- `autoindex` y `status` habilitados (riesgo).
- `mpm_prefork` en lugar de `mpm_event`.
- `php7_module` en lugar de PHP-FPM.

---

## 8. Configuración de PHP

| Directiva | Valor | Evaluación |
|-----------|-------|------------|
| display_errors | Off | Bien |
| memory_limit | -1 | Infinito |
| max_execution_time | 0 |  Infinito |
| post_max_size | 8M |  Bajo |
| upload_max_filesize | 2M | Muy bajo |

**Observaciones:**
- `memory_limit` y `max_execution_time` infinitos → riesgo de DoS.
- Límites de subida bajos → problemas para WordPress.

---







## 9. Firewall (UFW)

**Estado:** activo

| Puerto | Servicio | Restricción |
|--------|----------|-------------|
| 80 | HTTP | Anywhere |
| 443 | HTTPS | Anywhere |
| 2337 | SSH | Anywhere |
| 20/tcp | FTP datos | Anywhere |
| 21/tcp | FTP control | Anywhere |

**Observaciones:**
- No hay restricción por IP.
- FTP expuesto a internet.
- No hay rate limiting.

---

## 10. SSH

| Directiva | Valor | Evaluación |
|-----------|-------|------------|
| Port | 2337 | No estándar |
| PermitRootLogin | prohibit-password |  Mejor `no` |
| PasswordAuthentication | (comentada → yes) | Acepta contraseña |

**Observaciones:**
- SSH acepta login por contraseña (riesgo de fuerza bruta).
- Root puede entrar con clave SSH.

---

## 11. SSL / Certificados

| Elemento | Valor |
|----------|-------|
| Certificado | Wildcard *.undav.edu.ar |
| Emisor | Sectigo Limited |
| Válido desde | 5 ene 2026 |
| Válido hasta | 11 dic 2026 |
| Ubicación | /etc/certificados/certssl/undav.edu.ar.crt |
| Clave | /etc/certificados/certssl/undav.edu.ar.key |

**Observaciones:**
- **Vence el 11 dic 2026.**
- **No hay Certbot instalado** (no hay renovación automática).
- **No hay cron de renovación.**

---

## 12. Backups

**Script:** `/home/ggomez/backupftp/backup_base.sh`
**Directorio de backups:** `/home/ggomez/backupftp/datos/`

——————————————————————————————-



=== ETAPA 0 - ENTREGABLE 2 - 2026-10-14 ===
- WP-CLI instalado: /usr/local/bin/wp
- Versión: 2.12.0
- Funciona globalmente (probado desde /home y /tmp)
- Comandos de verificación: wp --version, wp --info
- Estado: COMPLETADO

——————————————————————————————————————————
=== ETAPA 0 - COMPLETADA - 2026-10-14 ===
- WordPress 7.1.3 instalado en /var/www/wptest.undav.edu.ar/
- Base de datos: wptest_db
- Usuario BD: wptest_user
- Usuario admin WP: mjlopez
- Contraseña admin WP: O80HzyO5JSQe
- Sitio: https://wptest.undav.edu.ar
- Panel: https://wptest.undav.edu.ar/wp-admin
- Tema activo: twentytwentyfive
- Plugins: akismet (inactivo), hello (inactivo)
- WP-CLI: 2.12.0
- Estado: COMPLETADO
