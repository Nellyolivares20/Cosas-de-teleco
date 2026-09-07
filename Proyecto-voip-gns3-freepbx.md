# Proyecto VoIP en GNS3: OSPF + VLANs + FreePBX + Zoiper

Documentación completa del proyecto: topología con routers OSPF (MD5), switches con VLANs y port-security, un servidor FreePBX corriendo en Docker dentro de GNS3, y un softphone Zoiper conectado desde el host Linux (CachyOS).

## 1. Topología final

```
[Zoiper en Linux host] --- tap0 --- Cloud1 --- SwGes-2 (VLAN 10/20) --- R3 --- R1 --- R2 --- SwGes-1 (VLAN 30/69) --- FreePBX (Docker)
                                                  g1/0        g2/0  g1/0    f0/0
```

| Segmento | Red |
|---|---|
| R1 - R2 | 192.168.0.0/30 |
| R1 - R3 | 192.168.4.0/30 |
| VLAN 10 (SwGes-2, Zoiper) | 192.168.10.0/24 |
| VLAN 20 (SwGes-2) | 192.168.20.0/24 |
| VLAN 30 (SwGes-1, FreePBX) | 192.168.30.0/24 |
| VLAN 69 (SwGes-1) | 192.168.69.0/24 |

- OSPF con autenticación **MD5** en el área 0, entre R1-R2 y R1-R3.
- Los routers R2 y R3 hacen **router-on-a-stick** (subinterfaces) para servir las VLANs de sus switches.
- SwGes-1 tiene **port-security** en el puerto del contenedor FreePBX: máximo 1 MAC, sticky, violation shutdown.

---

## 2. Instalación de paquetes en CachyOS (Arch-based)


### 2.2 Docker (para correr FreePBX)

```bash
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# cerrar sesión y volver a entrar para que el grupo tome efecto
```

### 2.3 libvirt (necesario para el nodo NAT de GNS3)

```bash
sudo pacman -S libvirt dnsmasq
sudo systemctl enable --now libvirtd
sudo virsh net-start default
sudo virsh net-autostart default
sudo usermod -aG libvirt $USER
```

### 2.4 tcpdump (diagnóstico de red)

```bash
sudo pacman -S tcpdump
```

### 2.5 Permisos de ubridge (crítico para que las interfaces de red de los nodos Docker funcionen)

Si los nodos Docker en GNS3 no reciben sus interfaces de red (`ip a` solo muestra `lo`), o ves el error:
```
Source NIO listener thread for bridge0 has stopped because of an error: Bad address
```
La solucion en el sgte comando:

```bash
sudo setcap cap_net_admin,cap_net_raw+ep $(which ubridge)
```
Después reiniciar GNS3 completo (cerar el proceso `gns3server` si se pega).

### 2.6 Zoiper (softphone)

Descargar el instalador oficial:
1. En `https://www.zoiper.com/en/voip-softphone/download/current` y bajar la versión Linux `.tar.xz`.
2. Extraer y ejecutar:
```bash
mkdir -p ~/zoiper5
tar -xJf ~/Downloads/Zoiper5_x.x.xx_x86_64.tar.xz -C ~/zoiper5
cd ~/zoiper5/Zoiper5
./zoiper
```

## 4. FreePBX en Docker dentro de GNS3

### 4.1 Imagen usada

```
flaviostutz/freepbx
```
Contenedor con FreePBX + Asterisk + MySQL en una sola imagen, lista para usar sin compilar nada.

### 4.2 Crear la plantilla del nodo Docker en GNS3

1. **Edit > Preferences > Docker > Docker containers > New**
2. Nombre de imagen: `flaviostutz/freepbx`
3. Adapters: 4 
4. Start command: **dejar vacío** (usa el `startup.sh` propio de la imagen)
5. En **Environment variables** (campo de texto al final de "General settings"), agregar:
```
ADMIN_PASSWORD= (la que sea, yo admin123)
FAIL2BAN_ENABLE=false
SIP_NAT_IP=192.168.30.10 (<- la ip por la que saldra y se conectara)
```

> `ADMIN_PASSWORD` es **obligatoria**, sin la contraseña el contenedor no arranca.
> `FAIL2BAN_ENABLE=false` evita un error de `iptables` (`can't initialize iptables table 'filter'`) que ocurre porque el kernel del host no tiene ciertos módulos cargados/porque el contenedor no corre en modo privilegiado con acceso a esas tablas.
> `SIP_NAT_IP=192.168.30.10` evita que el script de arranque (`apply-initial-configs.sh`) intente hacer `curl ifconfig.me` para autodetectar la IP pública — cosa que falla si el contenedor no tiene salida real a internet (como es el caso aquí, aislado en la VLAN 30). El script solo hace ese curl si la variable viene vacía; si se la pasamos, la usa directamente.

### 4.3 Conectar el nodo al switch

- Cablear `eth0` del contenedor al puerto de acceso VLAN 30 de SwGes-1 (Hay que crear el puerto para asignarle la ip)
- No hace falta ninguna otra interfaz conectada a internet una vez configurada la variable `SIP_NAT_IP`.

### 4.4 Iniciar y verificar

```bash
docker ps -a
```
Debe decir **Up**, no "Exited". Revisar el log de arranque:
```bash
docker logs -f <container_id>
```
Al final debe aparecer:
```
STARTUP COMPLETED
Waiting 3600 seconds for the next automatic backup...
```
Los mensajes `Can't find the module data for ...` durante la restauración del backup inicial son **normales** (warnings, no errores) en la primera inicialización.

### 4.5 Asignar IP dentro del contenedor

```bash
docker exec -it <container_id> bash, el id se extrae del (docker ps -a)
```
Dentro:
```bash(por defecto de la plantilla)
ip a                                 
ip addr add 192.168.30.10/24 dev eth0
ip route add default via 192.168.30.1
ping -c 4 192.168.30.1                 # probar llegada al gateway (R2)
exit
```

> Esta IP se pierde si el contenedor se reinicia y hay que volver a ingresar la ip.
### 4.6 Cómo encender y apagar el contenedor

**Desde GNS3 (recomendado, mantiene el cableado de la topología):**
- Clic derecho sobre el nodo → **Start** / **Stop** / **Suspend**.
- O usar el botón ▶️/⏹️ "Start all nodes" / "Stop all nodes" de la barra superior para toda la topología.

**Desde terminal (mismo contenedor Docker real, funciona igual):**
```bash
docker ps -a                 # ver el ID/estado actual
docker start <container_id>  # encender
docker stop <container_id>   # apagar limpio
docker restart <container_id>
```

> ⚠️ **Nunca uses la consola telnet turquesa de GNS3** (la ventana que abre al hacer doble clic sobre el nodo Docker) para escribir comandos esa consola está conectada a la salida estándar del proceso principal (`startup.sh`), no a una shell interactiva. Cualquier tecla que mandes ahí (incluso Enter) puede interpretarse como una señal y **matar el proceso** (código de salida 130 = SIGINT). Para entrar al contenedor, usa siempre `docker exec -it <id> bash` desde tu terminal normal.

## 5. Acceso al panel web de FreePBX y creación de extensiones

### 5.1 Acceder al panel

Desde cualquier equipo con ruta hacia la VLAN 30:
```
http://192.168.30.10 (IP asignada)
```

### 5.2 Asistente inicial ("Initial Setup")

- Username / Password: elegir uno propio.
- Notifications Email: cualquiera (no crítico para el lab).
- System Identifier: nombre libre.
- System Updates: recomendable dejar en **Disabled** (Automatic Module Updates y Security Updates) para un entorno de laboratorio sin salida a internet real.
- **Setup System**.

### 5.3 Crear una extensión PJSIP

**Applications > Extensions > Add Extension > Add New SIP [chan_pjsip] Extension**

- **User Extension**: número interno del "anexo" (ej. `6005`). Es el identificador corto con el que un softphone se registra y con el que otras extensiones lo pueden llamar directamente (como un anexo de oficina).
- **Display Name**: nombre descriptivo libre.
- **Secret**: contraseña SIP (se puede generar automática o poner una propia).
- **Submit** → luego **Apply Config** (botón rojo arriba; sin esto los cambios quedan guardados en base de datos pero Asterisk no los carga).

## 6. Conectar Zoiper (softphone) desde el host Linux

### 6.1 Crear una interfaz virtual dedicada (cloud)

```bash
sudo ip tuntap add dev tap0 mode tap
sudo ip link set tap0 up
```

### 6.2 Nodo Cloud en GNS3

1. Agregar un nodo **Cloud** en el canvas.
2. Configurarlo para usar la interfaz `tap0` (no `wlan0`, `docker0` ni `virbr0`).
3. Conectarlo al puerto de acceso VLAN 10 de SwGes-2.

### 6.3 Asignar IP en el host

```bash
sudo ip addr add 192.168.10.20/24 dev tap0
sudo ip route add 192.168.30.0/24 via 192.168.10.1
```

Probar conectividad:
```bash
ping 192.168.10.1     # gateway (R3)
ping 192.168.30.10    # FreePBX, a través de OSPF
```

### 6.4 Configurar la cuenta SIP en Zoiper

Al abrir Zoiper por primera vez **Accounts > Add**, elegir configuración **manual** (no auto-config):

| Campo | Valor |
|---|---|
| Usuario | `6005` (el User Extension creado en FreePBX) | (Elegir)
| Contraseña | el Secret de esa extensión |
| Dominio / Servidor / Host | `192.168.30.10` |
| Puerto | `5060` |
| Protocolo | SIP |

Si aparece un warning de "¿saltar la autodetección y configurar manualmente?" → **Yes**.

---

## 7. Diagnóstico de red usado durante el proyecto

### 7.1 Verificar vecindad OSPF

```
show ip ospf neighbor
```

### 7.2 Verificar estado de un puerto de switch

```
show interfaces status
show interfaces <puerto> switchport
show interfaces trunk
show spanning-tree vlan <n>
show vlan brief
```

### 7.3 Verificar interfaces y rutas dentro del contenedor FreePBX

```bash
docker exec -it <container_id> ip a
docker exec -it <container_id> ip route
docker exec -it <container_id> ping -c 4 <ip>
```

### 7.4 Verificar que Asterisk vea las extensiones y sus registros

```bash
docker exec -it <container_id> asterisk -rx "pjsip show endpoints"
docker exec -it <container_id> asterisk -rx "pjsip show registrations"
```

### 7.5 Capturar tráfico SIP para ver si los paquetes llegan

```bash
sudo tcpdump -i any port 5060 -n
```
(usar `-i tap0` o la interfaz específica si `any` da problemas de modo promiscuo)

---

## 8. Checklist obligatorio cada vez que se cierra y reabre el proyecto

Varias configuraciones **no son persistentes** y se pierden o revierten cada vez que se cierra GNS3 completo (o se reinician los nodos), aunque se haya hecho `write memory` o se hayan aplicado bien la primera vez. Repetir estos pasos en orden al reabrir:

### 8.1 Routers y switches
Verificar vecindad OSPF (por si tarda en converger tras el arranque):
```
show ip ospf neighbor
```

**Revisar SIEMPRE los puertos de acceso de ambos switches** — suelen volver a VLAN 1 por defecto al recargar, aunque el `write memory` se haya hecho correctamente antes:
```
show interfaces status
```
Si algún puerto de acceso aparece en VLAN 1 en vez de la VLAN que le corresponde, corregir:
```
configure terminal
interface <puerto>
 switchport access vlan <n>
end
write memory
```

### 8.2 Contenedor FreePBX (Docker)
La IP asignada manualmente a `eth0` no persiste. Reasignar:
```bash
docker exec -it <container_id> bash
ip addr add 192.168.30.10/24 dev eth0
ip route add default via 192.168.30.1
ping -c 4 192.168.30.1
exit
```

### 8.3 Interfaz tap0 / Cloud de GNS3 (lado Zoiper)
La interfaz `tap0` puede quedar en estado `DOWN` o perder su IP/ruta al reiniciar. Además, el link entre el nodo `Cloud1` y `SwGes-2` a veces necesita desconectarse y reconectarse manualmente en el canvas de GNS3 para que ubridge vuelva a pasar tráfico correctamente.

1. En GNS3: si el link entre `Cloud1` y `SwGes-2` no responde, eliminarlo y volver a conectarlo.
2. En el host Linux:
```bash
ip a show tap0                      # verificar si existe y su estado
sudo ip link set tap0 up            # si aparece DOWN
sudo ip addr add 192.168.10.20/24 dev tap0   # si perdió la IP
sudo ip route add 192.168.30.0/24 via 192.168.10.1   # si perdió la ruta
ping -c 4 192.168.10.1              # probar el primer salto (gateway R3)
ping -c 4 192.168.30.10             # probar FreePBX de punta a punta
```

> Si `ip addr add` o `ip route add` dan error de "File exists" o "RTNETLINK answers: File exists", significa que ese dato ya estaba puesto — no es un error real, se puede ignorar y seguir probando el ping.

### 8.4 Orden recomendado de verificación tras reabrir el lab:

1. `show ip ospf neighbor` en cualquier router.
2. `show interfaces status` en **ambos** switches — corregir VLANs de puertos de acceso si volvieron a VLAN 1.
3. Reasignar IP al contenedor FreePBX (`eth0`).
4. Revisar/recrear el link Cloud1 ↔ SwGes-2, y releventar `tap0` + IP + ruta en el host.
5. Probar `ping` de punta a punta (host Linux → `192.168.30.10`).
6. Abrir Zoiper y confirmar el ícono verde de registro (o forzar **Register** manualmente si quedó desconectado).
7. Probar `*43` (eco) para confirmar audio extremo a extremo.

---

## 9. Problemas encontrados y solución (bitácora de troubleshooting)

| Problema | Causa | Solución |
|---|---|---|
| `ADMIN_PASSWORD is required` | Falta variable de entorno obligatoria | Agregar `ADMIN_PASSWORD=admin123` en la plantilla Docker de GNS3 |
| `iptables v1.8.2: can't initialize iptables table 'filter'` | fail2ban intenta tocar el firewall del kernel sin módulos cargados | Agregar `FAIL2BAN_ENABLE=false` |
| `curl: (6) Could not resolve host: ifconfig.me` (contenedor muere, exit code 6) | El script de arranque intenta detectar la IP pública vía internet, pero el contenedor está aislado en la VLAN | Agregar variable de entorno `SIP_NAT_IP=192.168.30.10` (o editar el script directamente) |
| `Error while creating node from template: NAT interface virbr0 is missing, please install libvirt` | Falta libvirt para el nodo NAT de GNS3 | `sudo pacman -S libvirt dnsmasq`, activar `libvirtd` y la red `default` |
| `Source NIO listener thread for bridge0 has stopped because of an error: Bad address` / contenedor sin ninguna interfaz de red (`ip a` solo muestra `lo`) | `ubridge` sin permisos (capabilities) para manipular interfaces | `sudo setcap cap_net_admin,cap_net_raw+ep $(which ubridge)`, reiniciar GNS3 y recrear el link |
| GNS3 no abre, error `Could not connect to localhost on port 3080` | Quedó un proceso `gns3server` zombie de una sesión anterior ocupando el puerto | `ps aux \| grep gns3`, matar el PID viejo con `kill -9 <PID>`, reabrir GNS3 |
| Contenedor se cae con **Exited (130)** | Se envió una señal (tecla/Enter) desde la consola telnet de GNS3, que está conectada al proceso principal, no a una shell | No usar esa consola para interactuar; usar siempre `docker exec -it <id> bash` |
| `ping` a la VLAN dice "Destination Host Unreachable" | Puerto del switch hacia el router quedó en modo access/VLAN 1 en vez de trunk | Configurar `switchport mode trunk` + `switchport trunk allowed vlan <n>` en el puerto correcto |
| Igual que arriba pero con trunk ya bien configurado | Spanning Tree aún convergiendo, o VLAN no en estado forwarding | Esperar 30-50 seg. y verificar con `show spanning-tree vlan <n>` |
| Ping seguía fallando tras arreglar switch/STP | Typo en la IP de la subinterfaz del router (ej. `168.168.10.1` en vez de `192.168.10.1`) | Revisar con `show ip interface brief` y corregir la IP exacta |
| Zoiper: `401 Unauthorized` repetido | Contraseña (Secret) no coincide entre FreePBX y Zoiper | Copiar y pegar el Secret exacto desde el panel de FreePBX |
| Error de bridging al usar `wlan0` como interfaz del nodo Cloud | El WiFi no permite bridging de MACs ajenas (limitación 802.11) | Usar una interfaz `tap0` dedicada en vez de `wlan0` |
| Tras cerrar y reabrir el proyecto: puerto de acceso en el switch vuelve a VLAN 1, IP del contenedor se pierde, `tap0` queda DOWN sin IP/ruta | Ninguno de estos cambios manuales es persistente entre reinicios de los nodos | Ver checklist completo en la sección 8 — reasignar VLAN, IP del contenedor, y releventar `tap0` cada vez que se reabre el proyecto |

---

## 10. Comandos de referencia rápida

```bash
# Estado de Docker
docker ps -a
docker logs -f <id>
docker exec -it <id> bash
docker start <id>
docker stop <id>
docker restart <id>

# Estado de GNS3 / limpieza de procesos zombie
ps aux | grep gns3
kill -9 <PID>
gns3

# Diagnóstico de red
sudo tcpdump -i any port 5060 -n
sudo ss -tlnp | grep <puerto>

# libvirt / NAT
sudo virsh net-list --all
sudo virsh net-start default

# Permisos de ubridge
sudo setcap cap_net_admin,cap_net_raw+ep $(which ubridge)

# Interfaz TAP para Zoiper
sudo ip tuntap add dev tap0 mode tap
sudo ip link set tap0 up
sudo ip addr add 192.168.10.20/24 dev tap0
sudo ip route add 192.168.30.0/24 via 192.168.10.1
```

---

## 11. Resultado final

- Vecindad OSPF con autenticación MD5 formada correctamente entre R1-R2-R3.
- VLANs 10, 20, 30 y 69 funcionando con trunking y router-on-a-stick.
- Port-security activo en SwGes-1 (máximo 1 MAC, sticky, shutdown en violación).
- FreePBX corriendo en un contenedor Docker administrado por GNS3, integrado en la VLAN 30.
- Zoiper registrado exitosamente desde el host Linux (CachyOS) a través de una interfaz TAP conectada a la VLAN 10.
- Prueba de audio extremo a extremo confirmada con el test de eco (`*43`), validando que tanto la señalización SIP como el tráfico RTP de voz atraviesan correctamente toda la topología.
