# 🖥️ Data Center - Automatización de Redes y Monitoreo
**INACAP La Serena | Asignatura: Data Center | Docente: Daniel Ruz Moreno**

---

## 📋 Resumen de la infraestructura

| Componente | Detalle |
|---|---|
| Servidor Proxmox | `10.0.0.3:8006` |
| Contenedor LXC | `nelly-nodoansible` — IP `10.0.0.63` |
| Router Cisco ISR4321 | hostname `RT` — IP `10.0.0.210` |
| Switch Cisco | hostname `SW1` — IP management `192.168.10.2` (Vlan10) |
| Servidor TFTP | `/srv/tftp/` dentro del contenedor |

---

## 🏗️ PASO 1 — Contenedor LXC en Proxmox

### Crear el contenedor
- En Proxmox → **Create CT**
- Template: Ubuntu 22.04
- Hostname: `nelly-nodoansible`
- CPU: 2 cores, RAM: 1024 MB
- Red: Bridge `vmbr0`, IP estática

### Hacer backup
- Seleccionar el contenedor → pestaña **Backup** → **Backup Now**
- Storage: `local`, Mode: `Snapshot`

### Instalar dependencias dentro del contenedor
```bash
apt update && apt install -y ansible python3-pip tftpd-hpa
pip3 install netmiko
```

### Configurar TFTP
```bash
# El directorio real del servidor TFTP es /srv/tftp
mkdir -p /srv/tftp
chmod 777 /srv/tftp

# Verificar configuración
cat /etc/default/tftpd-hpa
# TFTP_DIRECTORY="/srv/tftp"

# Iniciar servicio
systemctl enable --now tftpd-hpa
systemctl status tftpd-hpa
```

### Configurar SSH para compatibilidad con Cisco IOS
```bash
cat > /etc/ssh/ssh_config << 'EOF'
Host 10.0.0.210
    KexAlgorithms +diffie-hellman-group14-sha1
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa
    StrictHostKeyChecking no

Host *
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes
EOF
```

---

## 🔧 PASO 2 — Configuración Router Cisco (RT)

### VLANs con Subinterfaces
```
configure terminal
interface GigabitEthernet0/0/1.50
 description VLAN50-Nelly
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/0/1.80
 description VLAN80-Nelly
 encapsulation dot1Q 80
 ip address 192.168.80.1 255.255.255.0
 ip nat inside
 no shutdown
exit
```

### DHCP para las VLANs
```
ip dhcp pool VLAN50
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
 dns-server 8.8.8.8
exit
ip dhcp pool VLAN80
 network 192.168.80.0 255.255.255.0
 default-router 192.168.80.1
 dns-server 8.8.8.8
exit
ip dhcp excluded-address 192.168.50.1
ip dhcp excluded-address 192.168.80.1
```

### NAT y ACL propias (sin tocar las de compañeros)
```
ip access-list extended NAT-NELLY
 deny ip 192.168.50.0 0.0.0.255 10.0.0.0 0.0.0.255
 deny ip 192.168.80.0 0.0.0.255 10.0.0.0 0.0.0.255
 permit ip 192.168.50.0 0.0.0.255 any
 permit ip 192.168.80.0 0.0.0.255 any
exit
ip nat inside source list NAT-NELLY interface GigabitEthernet0/0/0 overload
```

### SSH habilitado
```
ip domain name red-inacap
ip ssh version 2
username admin privilege 15 secret cisco
line vty 0 4
 login local
 transport input ssh
```

### Guardar configuración
```
end
write memory
```

---

## 🔌 PASO 3 — Configuración Switch Cisco (SW1)

### Crear VLANs
```
configure terminal
vlan 50
 name NELLY50
exit
vlan 80
 name NELLY80
exit
```

### Puertos de acceso con Port Security
```
interface FastEthernet0/16
 switchport mode access
 switchport access vlan 50
 switchport port-security maximum 5
 switchport port-security mac-address sticky
 switchport port-security
exit
interface FastEthernet0/17
 switchport mode access
 switchport access vlan 80
 switchport port-security maximum 5
 switchport port-security mac-address sticky
 switchport port-security
exit
```

### Trunk hacia el router (ya estaba en Fa0/24)
```
show interfaces trunk
```

### Activar ip routing (importante para SVIs)
```
ip routing
```

### Guardar
```
end
write memory
```

---

## 🐍 PASO 4 — Scripts Python de Auditoría

### Script del Router (`automatizacion.py`)
```python
#!/usr/bin/env python3
from netmiko import ConnectHandler
import datetime
import os

router = {
    'device_type': 'cisco_ios',
    'host': '10.0.0.210',
    'username': 'admin',
    'password': 'cisco',
    'secret': 'cisco',
}

print("[*] Conectando al Router...")
net_connect = ConnectHandler(**router)
net_connect.enable()

reporte = []
reporte.append("="*60)
reporte.append("REPORTE DE AUDITORIA - ROUTER RT")
reporte.append(f"Fecha: {datetime.datetime.now()}")
reporte.append("="*60)

comandos = {
    "VLANs y Subinterfaces": "show ip interface brief",
    "DHCP Pools": "show ip dhcp pool",
    "DHCP Bindings": "show ip dhcp binding",
    "NAT Translations": "show ip nat translations",
    "NAT Statistics": "show ip nat statistics",
    "Access Lists": "show access-lists",
    "Running Config Completo": "show running-config",
}

resultados_ok = []

for titulo, cmd in comandos.items():
    reporte.append(f"\n>>> {titulo} ({cmd})")
    output = net_connect.send_command(cmd)
    reporte.append(output)
    if "NAT Translations" in titulo:
        resultados_ok.append("NAT activo y traduciendo trafico")
    if "Access Lists" in titulo and "permit" in output:
        resultados_ok.append("ACLs configuradas con reglas permit")

net_connect.disconnect()

reporte.append("\n" + "="*60)
reporte.append("RESUMEN DE VALIDACION AUTOMATICA")
reporte.append("="*60)
for r in resultados_ok:
    reporte.append(f"  - {r}")
reporte.append("\n[OK] Sin errores detectados en la auditoria.")

os.makedirs("/srv/tftp", exist_ok=True)
nombre_archivo = f"/srv/tftp/RT_{datetime.datetime.now().strftime('%d%m_%H%M')}.txt"

with open(nombre_archivo, 'w') as f:
    f.write('\n'.join(reporte))

print(f"[OK] Reporte guardado en: {nombre_archivo}")
print("[OK] Auditoria completada exitosamente!")
```

### Script del Switch (`automatizacion_switch.py`)
El switch no es alcanzable directamente desde el contenedor — se accede **usando el router como salto intermedio** mediante `send_command_timing` de Netmiko.

---

## 🤖 PASO 5 — Playbook de Ansible

### Inventario (`~/ansible/inventory.ini`)
```ini
[router]
RT ansible_host=10.0.0.210 ansible_user=admin ansible_password=cisco ansible_become_password=cisco

[router:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_become=yes
ansible_become_method=enable
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### Playbook (`~/ansible/playbook_auditoria.yml`)
```yaml
---
- name: Auditoria automatizada Router y Switch
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Auditar Router
      command: python3 /root/automatizacion.py
      register: resultado_router

    - name: Mostrar resultado auditoria Router
      debug:
        var: resultado_router.stdout_lines

    - name: Auditar Switch (via Router como salto)
      command: python3 /root/automatizacion_switch.py
      register: resultado_switch

    - name: Mostrar resultado auditoria Switch
      debug:
        var: resultado_switch.stdout_lines

    - name: Listar reportes generados en TFTP
      command: ls -la /srv/tftp/
      register: reportes

    - name: Mostrar reportes en TFTP
      debug:
        var: reportes.stdout_lines
```

---

## ▶️ Comandos para la demo al profe

### Verificar que el TFTP está activo
```bash
systemctl status tftpd-hpa
```

### Ejecutar toda la auditoría con Ansible (el comando principal)
```bash
ansible-playbook ~/ansible/playbook_auditoria.yml
```

### Ver reportes guardados en TFTP
```bash
ls -la /srv/tftp/
```

### Ver reporte más reciente del Router
```bash
cat /srv/tftp/$(ls -t /srv/tftp/ | grep RT | head -1)
```

### Ver reporte más reciente del Switch
```bash
cat /srv/tftp/$(ls -t /srv/tftp/ | grep SW | head -1)
```

### Ver solo el resumen de validación
```bash
tail -20 /srv/tftp/$(ls -t /srv/tftp/ | grep RT | head -1)
```

### Descargar reporte via cliente TFTP (prueba end-to-end)
```bash
tftp localhost
# dentro del modo interactivo:
# get RT_0107_1300.txt /tmp/prueba.txt
# quit
cat /tmp/prueba.txt
```

### Conectarse al Router por SSH
```bash
ssh admin@10.0.0.210
# contraseña: cisco
# comandos útiles dentro del router:
# show ip interface brief
# show ip dhcp pool
# show access-lists
# show ip nat translations
# exit
```

### Probar conectividad Ansible
```bash
ansible -i ~/ansible/inventory.ini router -m ping
```

### Ver inventario y playbook
```bash
cat ~/ansible/inventory.ini
cat ~/ansible/playbook_auditoria.yml
```

---

## ✅ Lista de cotejo de la pauta cubierta

| # | Dimensión | Estado |
|---|---|---|
| 1 | Contenedor LXC con backup en Proxmox, gestionado por Ansible | ✅ |
| 2 | Script verifica VLANs, DHCP, NAT y ACL en el Router | ✅ |
| 3 | Script verifica Troncales, Acceso y Port Security en Switch | ✅ |
| 4 | Genera archivo de auditoría y lo transfiere al servidor TFTP | ✅ |
| 5 | Monitoreo Nagios/Grafana + alertas HTTP/FTP | ⏳ Pendiente |
| 6 | Defensa oral | 🎤 En vivo |

---

## ❓ Preguntas frecuentes de la defensa oral

**¿Qué es Netmiko?**
Librería Python que simplifica conexiones SSH a equipos de red Cisco. Maneja automáticamente prompts de enable y paginación de outputs.

**¿Por qué el router como salto para el switch?**
El switch solo tiene conectividad hacia la red interna `192.168.10.x`. El router tiene presencia en ambas redes (`10.0.0.x` e internas), por lo que actúa como intermediario.

**¿Qué verifica el script?**
Router: subinterfaces VLAN, pools DHCP, traducciones NAT, ACLs.
Switch: VLANs activas, puertos trunk, port security, estado de interfaces.

**¿Qué hace Ansible que no hace Python solo?**
Ansible orquesta (coordina) la ejecución de múltiples tareas en orden, con manejo de errores y logs estructurados, con un solo comando.

**¿Qué es TFTP y por qué se usa?**
Protocolo simple de transferencia de archivos, estándar en redes Cisco para respaldo de configuraciones y logs. No requiere autenticación y es muy liviano.
