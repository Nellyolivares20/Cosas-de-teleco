# 📺 Infraestructura TVHeadend + Active Directory con Ansible en AWS

> **Asignatura:** Data Center  
> **Carrera:** Ingeniería en Telecomunicaciones, Conectividad y Redes  
> **Instituto:** INACAP  
> **Docente:** Daniel Ruz Moreno  
> **Alumna:** Nelly Antinao Olivares
---

## 📋 Descripción del Proyecto

Despliegue de una infraestructura de red híbrida sobre **Amazon Web Services (AWS)** que incluye:

- Servidor de identidad con **Active Directory** y **GPOs** restrictivas
- Cliente Windows unido al dominio para validación de políticas
- Servidor Linux con **TVHeadend** instalado y configurado mediante **Ansible**
- Portal web **NellyTV** para visualización de canales IPTV

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│  PC LOCAL (Windows 11 + WSL2)                               │
│  • Ansible (nodo de control)                                │
│  • Cliente RDP para Windows Server y Client                 │
└─────────────────┬───────────────────────────────────────────┘
                  │ SSH / RDP
        ┌─────────┼──────────────────────┐
        │         │                      │
        ▼         ▼                      ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐
│ EC2          │ │ EC2          │ │ EC2                  │
│ Windows      │ │ Windows      │ │ Ubuntu 24.04         │
│ Server 2022  │ │ Client       │ │                      │
│              │ │              │ │ • TVHeadend          │
│ • AD DS      │ │ • Dominio    │ │ • Firewall UFW       │
│ • DNS        │ │  ServerNelly │ │ • Nginx              │
│ • GPOs       │ │ • GPO daniel │ │ • NellyTV Portal     │
│ • 6 Usuarios │ │              │ │                      │
└──────────────┘ └──────────────┘ └──────────────────────┘
  ServerNelly.com    (cliente)        54.166.192.192
```

---

## 🖥️ Instancias EC2 en AWS

| Nombre | SO | Tipo | Rol |
|--------|-----|------|-----|
| WNServerAdmin | Windows Server 2022 | m7i-flex.large (8 RAM) | Domain Controller |
| ClientePrueba | Windows 11 | m7i-flex.large | Cliente del dominio |
| Ansible | Ubuntu 24.04 | m7i-flex.large | TVHeadend + NellyTV |

### Puertos abiertos (Security Groups)

| Puerto | Protocolo | Servicio |
|--------|-----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP (NellyTV) |
| 3389 | TCP | RDP |
| 9981 | TCP | TVHeadend Web |
| 9982 | TCP | TVHeadend HTSP |
| 389 | TCP/UDP | LDAP (AD) |
| 53 | TCP/UDP | DNS (AD) |
| 88 | TCP/UDP | Kerberos (AD) |

---

## 🪟 Parte 1: Active Directory en Windows Server

### 1.1 Instalación de Roles

En **Server Manager → Add Roles and Features**:

- ✅ Active Directory Domain Services (AD DS)
- ✅ DNS Server

```
Reiniciar el servidor después de la instalación
```

### 1.2 Promover a Domain Controller

En Server Manager, click en la **bandera amarilla ⚠️** → "Promote this server to a domain controller":

| Campo | Valor |
|-------|-------|
| Tipo | Add a new forest |
| Root domain name | `ServerNelly.com` |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| DNS Server | ✅ Habilitado |
| Global Catalog | ✅ Habilitado |
| DSRM Password | `Inacap.2026!` |

```
El servidor se reinicia automáticamente al terminar
```

### 1.3 Estructura del Directorio

Abrir **Active Directory Users and Computers** → Tools:

```
ServerNelly.com
├── 📁 Ventas
│   ├── 👥 G_Ventas (Grupo de Seguridad Global)
│   ├── 👤 daniel  → Contraseña: Inacap2026#
│   └── 👤 maria   → Contraseña: Inacap2026#
├── 📁 TI
│   ├── 👥 G_TI (Grupo de Seguridad Global)
│   ├── 👤 carlos  → Contraseña: Inacap2026#
│   └── 👤 javiera → Contraseña: Inacap2026#
└── 📁 Administracion
    ├── 👥 G_Administracion (Grupo de Seguridad Global)
    ├── 👤 pedro   → Contraseña: Inacap2026#
    └── 👤 camila  → Contraseña: Inacap2026#
```

### 1.4 Crear Unidades Organizativas (UOs)

```
Click derecho en ServerNelly.com → New → Organizational Unit
Repetir para: Ventas, TI, Administracion
```

### 1.5 Crear Grupos de Seguridad

```
Dentro de cada UO → Click derecho → New → Group
Group scope: Global
Group type: Security
```

### 1.6 Crear Usuarios

```
Dentro de cada UO → Click derecho → New → User
✅ Password never expires
❌ User must change password at next logon
```

### 1.7 Agregar Usuarios a Grupos

```
Doble click en el grupo → Members → Add → escribir usuario → Check Names → OK
```

---

## 🔒 Parte 2: GPO Restrictiva para Ventas

### 2.1 Crear y Enlazar la GPO

Abrir **Group Policy Management** → Tools:

```
Click derecho en UO "Ventas" → Create a GPO in this domain and Link it here
Nombre: GPO_Restriccion_Ventas
```

### 2.2 Editar la GPO

```
Click derecho en GPO_Restriccion_Ventas → Edit
```

#### Bloquear Panel de Control ⭐ (CRÍTICO)

```
User Configuration
→ Policies
→ Administrative Templates
→ Control Panel
→ "Prohibit access to Control Panel and PC Settings"
→ Estado: Enabled
```

#### Solo permitir Edge (Run only specified apps)

```
User Configuration
→ Policies
→ Administrative Templates
→ System
→ "Run only specified Windows applications"
→ Estado: Enabled
→ List of allowed applications → Show:
   msedge.exe
```

#### Forzar Edge en escritorio (Desktop Persistence)

```
User Configuration
→ Preferences
→ Windows Settings
→ Shortcuts
→ New → Shortcut
   Action: Update
   Name: Microsoft Edge
   Location: Desktop
   Target: C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
```

#### Configuraciones adicionales de Desktop

```
User Configuration → Policies → Administrative Templates → Desktop
✅ Remove the Desktop Cleanup Wizard → Enabled
✅ Don't save settings at exit → Enabled
```

### 2.3 Aplicar la GPO

En el **Windows Server**, PowerShell como administrador:

```powershell
gpupdate /force
```

---

## 💻 Parte 3: Unir Cliente Windows al Dominio

### 3.1 Configurar DNS en el cliente

```
Settings → Network & Internet → Ethernet → Edit (DNS)
Preferred DNS: 172.31.33.18  (IP privada del Windows Server)
Alternate DNS: 8.8.8.8
```

### 3.2 Habilitar comunicación entre instancias

En el **Windows Server**, PowerShell como administrador:

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

En el **Windows Client**, PowerShell como administrador:

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

### 3.3 Unir al dominio

```powershell
Add-Computer -DomainName "ServerNelly.com" -Credential (Get-Credential) -Restart -Force
```

Credenciales:
- Usuario: `Administrator`
- Contraseña: `Inacap2026#`

### 3.4 Habilitar RDP para usuarios del dominio

```powershell
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "SERVERNELLY\daniel"
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### 3.5 Verificar GPO con usuario daniel

```
Login: SERVERNELLY\daniel
Password: Inacap2026#
```

**Resultado esperado:**
- ✅ Edge forzado en el escritorio
- ❌ Panel de Control bloqueado
- ❌ Otras aplicaciones bloqueadas

---

## 🤖 Parte 4: Ansible - Nodo de Control en WSL2

### 4.1 Instalar WSL2 con Ubuntu 22.04

En **PowerShell como administrador**:

```powershell
wsl --install -d Ubuntu-22.04
```

Reiniciar el PC y crear usuario/contraseña en Ubuntu.

### 4.2 Instalar Ansible

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
ansible --version
```

### 4.3 Configurar llave SSH

```bash
mkdir -p ~/.ssh
cp "/mnt/c/Users/Nellyyy/Downloads/Acces ansible.pem" ~/.ssh/acces_ansible.pem
chmod 400 ~/.ssh/acces_ansible.pem
```

### 4.4 Crear carpeta del proyecto

```bash
mkdir -p ~/proyecto-tvheadend
cd ~/proyecto-tvheadend
```

### 4.5 Crear inventario `hosts.ini`

```bash
nano hosts.ini
```

```ini
[servidores_iptv]
54.166.192.192 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/acces_ansible.pem ansible_python_interpreter=/usr/bin/python3
```

### 4.6 Verificar conectividad

```bash
ansible servidores_iptv -i hosts.ini -m ping
```

**Resultado esperado:**
```json
54.166.192.192 | SUCCESS => {
    "ping": "pong"
}
```

---

## 📜 Parte 5: Playbook Ansible - `tvheadend.yml`

### 5.1 Crear el playbook

```bash
nano tvheadend.yml
```

```yaml
---
- name: Despliegue completo TVHeadend con Firewall y SSH
  hosts: servidores_iptv
  become: yes

  tasks:

    # ==================
    # ACTUALIZAR SISTEMA
    # ==================
    - name: Actualizar cache de paquetes APT
      apt:
        update_cache: yes
        cache_valid_time: 3600

    # ==================
    # INSTALAR TVHEADEND
    # ==================
    - name: Instalar snapd
      apt:
        name: snapd
        state: present

    - name: Asegurar que snapd este activo
      systemd:
        name: snapd
        state: started
        enabled: yes

    - name: Instalar TVHeadend mediante Snap
      community.general.snap:
        name: tvheadend
        state: present

    - name: Verificar estado de TVHeadend
      shell: snap services tvheadend
      register: tvh_status
      changed_when: false

    - name: Mostrar estado TVHeadend
      debug:
        var: tvh_status.stdout_lines

    # ==================
    # CONFIGURAR FIREWALL
    # ==================
    - name: Instalar UFW
      apt:
        name: ufw
        state: present

    - name: Resetear reglas UFW
      ufw:
        state: reset

    - name: Politica por defecto - denegar entrada
      ufw:
        default: deny
        direction: incoming

    - name: Politica por defecto - permitir salida
      ufw:
        default: allow
        direction: outgoing

    - name: Permitir SSH (puerto 22)
      ufw:
        rule: allow
        port: '22'
        proto: tcp
        comment: 'SSH acceso remoto'

    - name: Permitir HTTP (puerto 80)
      ufw:
        rule: allow
        port: '80'
        proto: tcp
        comment: 'Portal NellyTV'

    - name: Permitir TVHeadend Web (puerto 9981)
      ufw:
        rule: allow
        port: '9981'
        proto: tcp
        comment: 'TVHeadend interfaz web'

    - name: Permitir TVHeadend HTSP (puerto 9982)
      ufw:
        rule: allow
        port: '9982'
        proto: tcp
        comment: 'TVHeadend streaming HTSP'

    - name: Habilitar UFW
      ufw:
        state: enabled

    - name: Verificar reglas UFW
      shell: ufw status verbose
      register: ufw_status
      changed_when: false

    - name: Mostrar reglas del firewall
      debug:
        var: ufw_status.stdout_lines

    # ==================
    # CONFIGURAR SSH
    # ==================
    - name: Asegurar que OpenSSH este instalado
      apt:
        name: openssh-server
        state: present

    - name: Deshabilitar login root por SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        backup: yes

    - name: Deshabilitar autenticacion por password
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'

    - name: Habilitar autenticacion por llave publica
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PubkeyAuthentication'
        line: 'PubkeyAuthentication yes'

    - name: Configurar timeout SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?ClientAliveInterval'
        line: 'ClientAliveInterval 300'

    - name: Configurar max intentos SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?MaxAuthTries'
        line: 'MaxAuthTries 3'

    - name: Reiniciar servicio SSH
      systemd:
        name: ssh
        state: restarted
        enabled: yes

    - name: Verificar configuracion SSH
      shell: sshd -T | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries"
      register: ssh_config
      changed_when: false

    - name: Mostrar configuracion SSH aplicada
      debug:
        var: ssh_config.stdout_lines

    # ==================
    # INSTALAR NGINX + PHP
    # ==================
    - name: Instalar Nginx
      apt:
        name: nginx
        state: present

    - name: Instalar PHP y PHP-FPM
      apt:
        name:
          - php
          - php-fpm
        state: present

    - name: Asegurar que Nginx este activo
      systemd:
        name: nginx
        state: started
        enabled: yes

    - name: Asegurar que PHP-FPM este activo
      systemd:
        name: php8.3-fpm
        state: started
        enabled: yes

    # ==================
    # RESUMEN FINAL
    # ==================
    - name: Resumen de servicios activos
      shell: |
        echo "=== TVHEADEND ===" && snap services tvheadend
        echo "=== FIREWALL ===" && ufw status
        echo "=== NGINX ===" && systemctl is-active nginx
        echo "=== PHP-FPM ===" && systemctl is-active php8.3-fpm
        echo "=== SSH ===" && systemctl is-active ssh
      register: resumen
      changed_when: false

    - name: Mostrar resumen final
      debug:
        var: resumen.stdout_lines
```

### 5.2 Verificar sintaxis

```bash
ansible-playbook -i hosts.ini tvheadend.yml --syntax-check
```

### 5.3 Ejecutar el playbook

```bash
ansible-playbook -i hosts.ini tvheadend.yml
```

### 5.4 Resultado esperado

```
PLAY RECAP *******************************************
54.166.192.192 : ok=X   changed=X   unreachable=0   failed=0
```

---

## 📺 Parte 6: Configuración de TVHeadend

### 6.1 Acceder a la interfaz web

```
http://IP_PUBLICA:9981
```

### 6.2 Asistente de configuración

| Campo | Valor |
|-------|-------|
| Idioma interfaz | Spanish |
| Red permitida | 0.0.0.0/0 |
| Usuario admin | admin |
| Contraseña admin | inacap |
| Usuario regular | alumno |
| Contraseña regular | inacap |

### 6.3 Configurar red IPTV

```
Configuración → Entradas DVB → Redes → Añadir
Tipo: Red automática IPTV
URL: https://iptv-org.github.io/iptv/countries/cl.m3u
```

### 6.4 Mapear canales

```
Configuración → Canales/EPG → Mapear servicios a canales
✅ Mapear todos los servicios
```

---

## 🌐 Parte 7: Portal NellyTV

### 7.1 Instalar dependencias (ya hecho por Ansible)

```bash
sudo apt install nginx php php-fpm -y
```

### 7.2 Crear proxy PHP

```bash
sudo nano /var/www/html/proxy.php
```

```php
<?php
header('Access-Control-Allow-Origin: *');
header('Content-Type: text/plain');

$urls = [
    'https://iptv-org.github.io/iptv/countries/cl.m3u',
    'https://iptv-org.github.io/iptv/countries/ar.m3u',
];

$opts = ['http' => ['header' => "User-Agent: Mozilla/5.0\r\n"]];
$context = stream_context_create($opts);

$combined = "#EXTM3U\n";
foreach ($urls as $url) {
    $content = file_get_contents($url, false, $context);
    if ($content) {
        foreach (explode("\n", $content) as $line) {
            if (!str_starts_with(trim($line), '#EXTM3U')) {
                $combined .= $line . "\n";
            }
        }
    }
}
echo $combined;
?>
```

### 7.3 Configurar Nginx para PHP

```bash
nano /etc/nginx/sites-enabled/default
```

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    index index.php index.html index.htm;
    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
}
```

```bash
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl restart php8.3-fpm
```

### 7.4 Acceder al portal

```
http://IP_PUBLICA
```

**Credenciales:**

| Usuario | Contraseña | Rol |
|---------|-----------|-----|
| tv | lascochinas123 | Usuario principal |
| admin | inacap | Administrador |
| daniel | Inacap2026#| Usuario dominio |

---



## 👩‍💻 Autora

**Nelly Antinao Okivares** — Ingeniería en Telecomunicaciones, Conectividad y Redes  
INACAP La Serena — 2026

---


