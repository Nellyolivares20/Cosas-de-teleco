# ☁️ AWS Cloud Lab: Active Directory & Web Server con Control Delegado

Este repositorio documenta el despliegue de una infraestructura híbrida en **Amazon Web Services (AWS)** utilizando **Windows Server 2022**. El objetivo principal es la implementación de un dominio centralizado y la delegación de permisos de administración web a un usuario específico mediante políticas NTFS.

## 🏗️ Arquitectura del Proyecto

La topología de red se compone de dos instancias **EC2** dentro de una misma VPC:

* **Servidor Admin (Domain Controller):** Actúa como el cerebro de la red, gestionando identidades y servicios DNS.
* **Servidor Cliente (Web Server):** Host del servicio IIS que se une al dominio para recibir políticas de seguridad centralizadas.

| Recurso | Sistema Operativo | Rol Principal | IP Privada (Ejemplo) |
| :--- | :--- | :--- | :--- |
| **Admin** | Windows Server 2022 | AD DS / DNS | `172.31.75.58` |
| **Cliente** | Windows Server 2022 | IIS / Web Server | `172.31.xx.xx` |

---

## 🛠️ Paso 1: Preparación en AWS

Antes de la configuración interna, se preparó el entorno de nube:

1. **Security Groups:** Se configuraron reglas de entrada (Inbound Rules) para permitir:
   * **RDP (3389):** Para administración remota.
   * **HTTP (80):** Para el acceso al servidor web.
   * **DNS (UDP/TCP 53):** Crucial para la comunicación entre instancias.
   * *Nota: Se recomienda "All Traffic" para entornos de prueba internos entre ambas IPs.*
2. **Acceso:** Se realizó el descifrado de contraseñas mediante la consola de AWS utilizando el par de claves `.pem` generado al crear las instancias.

---

## 🔑 Paso 2: Configuración del Controlador de Dominio (Admin)

Se transformó la instancia Admin en un servidor maestro:

1. **Instalación de Roles:** A través de *Server Manager*, se instaló el rol de **Active Directory Domain Services (AD DS)**.
2. **Promoción del Servidor:** Se promovió la instancia a Controlador de Dominio creando un nuevo bosque llamado `Prueba.com`.
3. **Gestión de Usuarios:** En *Active Directory Users and Computers*, se creó el usuario `daniel` dentro del contenedor de `Users`.

```powershell
# Ejemplo de verificación de dominio vía PowerShell
Get-ADDomain -Identity Prueba.com
```

---

## 🔌 Paso 3: Conexión del Servidor Cliente

Para que el cliente reconozca al dominio, se debe redirigir su "brújula" de red:

1. **Configuración DNS:** En las propiedades de red (IPv4) del Servidor Cliente, se asignó manualmente la IP privada del Admin (`172.31.75.58`) como servidor DNS preferido.
2. **Unión al Dominio:** Se cambió el grupo de trabajo por el dominio `Prueba.com`.
3. **Autenticación:** Se utilizaron credenciales de administrador del dominio para autorizar la unión y se reinició el sistema.

---

## 🌐 Paso 4: Despliegue del Servidor Web (IIS)

Con la máquina ya integrada en el dominio:

1. **Instalación de IIS:** Se habilitó el rol de *Web Server (IIS)* en el Servidor Cliente.
2. **Contenido Base:** Se navegó a la ruta raíz del servidor: `C:\inetpub\wwwroot`.
3. **Archivo Index:** Se generó un archivo `index.html` básico para las pruebas de edición.

---

## 🛡️ Paso 5: Control Delegado (Permisos NTFS)

Este es el núcleo de la seguridad del proyecto. Se aplicó el principio de "menor privilegio":

1. **Deshabilitar Herencia:** En las propiedades de seguridad de `index.html`, se desactivó la herencia para romper los permisos por defecto del sistema.
2. **Limpieza de Grupos:** Se eliminó el acceso de escritura al grupo general "Users".
3. **Asignación Específica:** Se agregó al usuario `PRUEBA\daniel` otorgándole explícitamente permisos de:
   * *Read (Lectura)*
   * *Write (Escritura)*
   * *Modify (Modificación)*

---

## ✅ Paso 6: Verificación y Pruebas de Acceso

Para validar que el control delegado funciona correctamente:

1. **Login Remoto:** Se accedió al Servidor Cliente vía RDP usando las credenciales del dominio: `User: PRUEBA\daniel`.
2. **Autorización RDP:** (Paso previo) Se agregó a Daniel al grupo de "Remote Desktop Users" en la configuración de sistema del cliente.
3. **Prueba de Edición:**
   * Se abrió el archivo `index.html` con el Bloc de Notas.
   * Se realizaron cambios y se guardaron exitosamente.
4. **Validación de Negación:** Se comprobó que otros usuarios sin permisos específicos no pueden sobrescribir el archivo, garantizando la integridad del sitio.

---

> **Skills demostradas:** Cloud Computing (AWS), SysAdmin (Windows Server), Networking (DNS/TCP-IP), Identidad (Active Directory) y Ciberseguridad (NTFS Permissions).
