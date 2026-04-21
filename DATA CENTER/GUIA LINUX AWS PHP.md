# 🚀 Guía Maestra: Despliegue de Servidor Web en AWS

> **Proyecto:** Agenda Central Alameda 💎
> **Servidor:** Ubuntu Server en AWS EC2
> **Stack:** LAMP (Linux, Apache, MySQL, PHP)

Este documento sirve como **manual de referencia rápido** y como **base de conocimientos** para que cualquier IA o colaborador entienda exactamente cómo está montada la infraestructura.

---

## 📑 Tabla de Contenidos

- [Fase 1: Configuración de la Instancia (AWS)](#️-fase-1-configuración-de-la-instancia-aws)
- [Fase 2: Conexión y Gestión de Linux](#-fase-2-conexión-y-gestión-de-linux)
- [Fase 3: Instalación del Stack LAMP](#-fase-3-instalación-del-stack-lamp)
- [Fase 4: Administración de Archivos Web](#-fase-4-administración-de-archivos-web)
- [Fase 5: Mantenimiento y Rendimiento](#️-fase-5-mantenimiento-y-rendimiento)
- [Notas de Seguridad del Administrador](#️-notas-de-seguridad-del-administrador)

---

## 🏗️ Fase 1: Configuración de la Instancia (AWS)

1. **Instancia:** Crear instancia Ubuntu Server en AWS EC2.
2. **Seguridad (Keys):** Generar par de claves RSA (`.pem`).
   > ⚠️ **Nota:** Guardar el archivo `.pem` en un lugar seguro. No se puede volver a descargar.
3. **Red:** Habilitar IP Pública automática.
4. **Firewall (Security Groups):** Abrir **Todo el tráfico (TCP Personalizado)** con origen `0.0.0.0/0` (Anywhere IPv4) para garantizar conectividad total durante el desarrollo.

---

## 💻 Fase 2: Conexión y Gestión de Linux

### Acceso SSH

```bash
ssh -i "TuClave.pem" ubuntu@TU_IP_PUBLICA
```

### Comandos Esenciales de Navegación

| Comando | Descripción |
|---------|-------------|
| `pwd` | ¿Dónde estoy? (imprime ruta actual) |
| `ls` | ¿Qué hay aquí? (lista archivos y carpetas) |
| `cd /ruta` | Moverse entre carpetas |
| `sudo su` | Entrar como Administrador (Root) |

---

## 📦 Fase 3: Instalación del Stack LAMP

### 1. Actualización del sistema e instalación de MySQL

```bash
sudo apt-get update
sudo apt-get install mysql-server
```

### 2. Instalación de PHP y Apache

```bash
sudo apt install php libapache2-mod-php php-mysql
sudo systemctl restart apache2   # Refrescar el servidor
```

### 3. Configuración de Base de Datos (MySQL)

Entrar a MySQL:

```bash
sudo mysql -u root -p
```

Crear base de datos, usuario y tabla principal:

```sql
CREATE DATABASE central_alameda;
CREATE USER 'admin_central'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Atecentral206';
GRANT ALL PRIVILEGES ON central_alameda.* TO 'admin_central'@'localhost';
FLUSH PRIVILEGES;

USE central_alameda;

CREATE TABLE servicios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    patente VARCHAR(15),
    nombre_cliente VARCHAR(100),
    telefono_cliente VARCHAR(20),
    modelo_auto VARCHAR(50),
    producto_instalar VARCHAR(100),
    observaciones TEXT,
    observacion_trabajador TEXT,
    foto_entrega VARCHAR(255),
    llego_al_taller TINYINT(1) DEFAULT 0,
    estado_pago TINYINT(1) DEFAULT 0,
    reporte_sellado INT DEFAULT 0, -- Agregada para el candado del taller
    fecha_ingreso DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 📂 Fase 4: Administración de Archivos Web

El corazón de la página reside en `/var/www/html/`.

| Acción | Comando |
|--------|---------|
| Editar el código | `sudo nano /var/www/html/index.php` |
| Ver el código | `cat /var/www/html/index.php` |
| Borrar el archivo | `sudo rm /var/www/html/index.php` |
| Vaciar archivo rápido | `sudo truncate -s 0 /var/www/html/index.php` |
| Reiniciar el servidor web | `sudo systemctl restart apache2` |

---

## 🛠️ Fase 5: Mantenimiento y Rendimiento

- **Monitoreo:** Usar `htop` para ver CPU y RAM en tiempo real.
- **Caché del navegador:** Si la web no muestra cambios, usar `Ctrl + F5` o limpiar caché con `Ctrl + Shift + Del`.
- **Rutas importantes:**
  - `/etc/mysql/mysql.conf.d/mysqld.cnf` → Configuración profunda de la base de datos.
  - `/var/www/html/` → Archivos del sitio web.
  - `/var/www/html/fotos_autos/` → Carpeta de imágenes subidas.

---

## 🛡️ Notas de Seguridad del Administrador

> ⚠️ **IMPORTANTE**

1. **Usuario de base de datos:** `admin_central` con la clave definida durante la configuración.
2. **Carpeta de fotos:** Las imágenes se guardan en `/var/www/html/fotos_autos/`. Esta carpeta debe tener permisos:
   ```bash
   sudo chmod 777 /var/www/html/fotos_autos/
   ```
3. **Error 500 después de editar:** Si el servidor falla con un Error 500 luego de modificar archivos, lo más probable es un **error de sintaxis en `index.php`**. Revisa el archivo o consulta los logs:
   ```bash
   sudo tail -f /var/log/apache2/error.log
   ```

---

## 📌 Resumen Técnico

| Componente | Valor |
|------------|-------|
| **Sistema Operativo** | Ubuntu Server (AWS EC2) |
| **Servidor Web** | Apache 2 |
| **Motor de BD** | MySQL |
| **Lenguaje** | PHP |
| **Base de Datos** | `central_alameda` |
| **Usuario DB** | `admin_central` |
| **Directorio Web** | `/var/www/html/` |
| **Directorio de Imágenes** | `/var/www/html/fotos_autos/` |

---

_Documento generado como guía interna para el proyecto **Central Alameda**._
