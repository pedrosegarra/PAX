# Solucionario del Profesor

Este documento contiene las configuraciones en línea de comandos (export) para dar respuesta a los tres ejercicios y retos planteados en el curso. 

**Disclaimer:** En MikroTik existen múltiples formas de resolver un mismo problema (por ejemplo, usando listas de interfaces `interface-list` en lugar de nombrar interfaz por interfaz). Estas son soluciones modelo.

---

## 1. Solución: Tarea Bloque 4 (Firewall Básico y Redirección)

Solucionario referenciado sobre el archivo `docs/bloque4/tarea_bloque4.md`.

```routeros
# Fase 1. Enrutamiento y NAT
/ip address add address=192.168.10.1/24 interface=ether2
/ip address add address=172.16.20.1/24 interface=ether3
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

# Fase 2. Reglas de Protección (Input)
/ip firewall filter add chain=input action=accept connection-state=established,related comment="Aceptar tráfico establecido/relacionado"

/ip firewall address-list add list=Admin_IPs address=192.168.10.50
/ip firewall filter add chain=input action=accept src-address-list=Admin_IPs comment="Permitir Admin"

/ip firewall filter add chain=input action=drop comment="Bloquear todo lo demas hacia el router"

# Fase 3. Redirección de Puertos (Port Forwarding para DMZ)
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=172.16.20.10 to-ports=80 protocol=tcp dst-port=80 in-interface=ether1 comment="Publicar HTTP DMZ al exterior"
```

---

## 2. Solución: Tarea Bloque 5 (VLANs + Políticas L3)

Solucionario referenciado sobre el archivo `docs/bloque5/tarea_bloque5.md`.

```routeros
# Fase 1. Despliegue de VLANs y Capa 3
/interface bridge add name=bridge-trunk
/interface bridge port add bridge=bridge-trunk interface=ether2

/interface vlan add name=vlan10_admin interface=bridge-trunk vlan-id=10
/interface vlan add name=vlan20_profes interface=bridge-trunk vlan-id=20
/interface vlan add name=vlan30_alumnos interface=bridge-trunk vlan-id=30

/ip address add address=192.168.10.1/24 interface=vlan10_admin
/ip address add address=192.168.20.1/24 interface=vlan20_profes
/ip address add address=192.168.30.1/24 interface=vlan30_alumnos

# Fase 2. Servicios (Se asume creación mediante ip dhcp-server setup o script completo)
# Ejemplo de configuración para Admin (repetir equivalentes para Profesores y Alumnos)
/ip pool add name=pool_admin ranges=192.168.10.10-192.168.10.100
/ip dhcp-server add name=dhcp_admin interface=vlan10_admin address-pool=pool_admin
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=8.8.8.8

/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1

# Fase 3. Seguridad mediante Firewall (Forwarding restringido)
/ip firewall filter add chain=forward action=accept connection-state=established,related comment="Permitir respuestas"

/ip firewall filter add chain=forward action=accept src-address=192.168.20.0/24 dst-address=192.168.10.0/24 comment="Profesores -> Admin"
/ip firewall filter add chain=forward action=accept src-address=192.168.10.0/24 dst-address=192.168.20.0/24 comment="Admin -> Profesores"

/ip firewall filter add chain=forward action=drop src-address=192.168.30.0/24 dst-address=192.168.10.0/24 comment="Bloqueo Alumnos a Admin"
/ip firewall filter add chain=forward action=drop src-address=192.168.30.0/24 dst-address=192.168.20.0/24 comment="Bloqueo Alumnos a Profesores"
# Opcionalmente se podría invertir la lógica y descartar todo tráfico local, y permitir solo la inter-vlan autorizada.
```

---

## 3. Solución: Reto Integrador (Proyecto Final)

Solucionario referenciado sobre el archivo `docs/proyecto_final/reto_integrador.md`.

```routeros
# 1. Configuración de Conectividad Base
/ip dhcp-client add interface=ether1 disabled=no

# Bridge y VLAN Filtering para el brazo de la LAN
/interface bridge add name=bridge-lan vlan-filtering=yes
/interface bridge port add bridge=bridge-lan interface=ether2
/interface vlan add name=vlan100_admin interface=bridge-lan vlan-id=100
/interface vlan add name=vlan200_opers interface=bridge-lan vlan-id=200
/interface bridge vlan add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=100,200

# Capa IP de Puertas de Enlace
/ip address add address=10.10.100.1/24 interface=vlan100_admin
/ip address add address=10.10.200.1/24 interface=vlan200_opers
/ip address add address=172.16.50.1/24 interface=ether3

# Resolución DNS
/ip dns set allow-remote-requests=yes

# Servidores DHCP limitados a .50 - .100 (Ejemplo para VLAN100. Clonar para VLAN200)
/ip pool add name=pool_v100 ranges=10.10.100.50-10.10.100.100
/ip dhcp-server add name=dhcp_v100 interface=vlan100_admin address-pool=pool_v100
/ip dhcp-server network add address=10.10.100.0/24 gateway=10.10.100.1 dns-server=10.10.100.1

# 2. Controles Lógicos y Seguridad

# Crear listas lógicas para agrupar interfaces
/interface list add name=WAN
/interface list add name=LAN
/interface list member add list=WAN interface=ether1
/interface list member add list=LAN interface=vlan100_admin
/interface list member add list=LAN interface=vlan200_opers

# Masquerade (SNAT) exclusivo para la red corporativa al salir por la WAN
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN src-address=10.10.0.0/16

# Firewall INPUT (Proteger al Router)
/ip firewall filter add chain=input action=accept connection-state=established,related
/ip firewall filter add chain=input action=accept protocol=icmp in-interface-list=WAN comment="Permitir ICMP desde WAN"
/ip firewall filter add chain=input action=accept in-interface=vlan100_admin dst-port=8291 protocol=tcp comment="Permitir Winbox desde Admin"
/ip firewall filter add chain=input action=drop in-interface-list=WAN comment="Bloquear todo lo demás desde Internet al Router"
/ip firewall filter add chain=input action=drop in-interface=vlan200_opers dst-port=8291 protocol=tcp comment="Bloquear acceso de Operadores al Winbox"

# Firewall FORWARD (Tráfico pasante)
/ip firewall filter add chain=forward action=accept connection-state=established,related
/ip firewall filter add chain=forward action=drop src-address=172.16.50.0/24 dst-address=10.10.0.0/16 comment="Aislamiento: Bloquear inicio de conexión desde DMZ hacia LAN"

# Port Forwarding al Servidor Web
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=172.16.50.5 to-ports=80 protocol=tcp dst-port=80 in-interface-list=WAN
```
