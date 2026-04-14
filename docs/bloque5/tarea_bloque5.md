# Tarea: Segmentación con VLANs y políticas de Firewall

## Objetivo
Implementar un entorno de múltiples subredes enrutadas mediante el uso de VLANs sobre un bridge común, y restringir la comunicación inter-VLAN haciendo uso del firewall.

## Descripción del Escenario

En una institución educativa te solicitan rediseñar su arquitectura de red usando un único router MikroTik y un despliegue de Switches de Capa 2 transparente. Todos los dispositivos de los distintos departamentos llegarán por un mismo enlace de agregación (Trunk) al puerto `ether2` de tu router.

Las VLANs que intervienen en tu puerto troncal son:
- VLAN 10 – Administración (`192.168.10.0/24`)
- VLAN 20 – Profesores (`192.168.20.0/24`)
- VLAN 30 – Alumnos (`192.168.30.0/24`)

## Tareas a Realizar

### Fase 1. Despliegue de VLANs y L3
- Crea un interface de tipo **Bridge** llamado `bridge-trunk`.
- Añade el puerto troncal (`ether2`) a ese bridge.
- Crea las correspondientes interfaces VLAN (VLAN 10, 20 y 30) cuyo *interface* base sea el `bridge-trunk`.
- Configura la primera IP utilizable de cada rango a modo de Gateway para cada interfaz (ej. `192.168.10.1` para la VLAN 10).

### Fase 2. Red y Servicios
- Aprovisiona tres servidores DHCP independientes en el router, uno para la subred de cada VLAN. Comprueba desde máquinas cliente de cada subred que obtienen correctamente IPs diferentes.
- Configura NAT dinámico (masquerade) hacia la WAN para que todas las VLAN tengan salida a Internet.

### Fase 3. Seguridad mediante Firewall (Forwarding)
Por defecto, al pertenecer todas a routers conectadas por el mismo equipo, existe encaminamiento entre todas. Nos solicitan limitarlo por seguridad:
1. Permite explícitamente el tráfico entre la **VLAN Profesores** y la **VLAN Administración**.
2. Los dispositivos de la **VLAN Alumnos** no deben tener permiso para comunicarse con ninguna de las otras dos VLANs internas, pero SÍ deben poder acceder a Internet.
3. Permite siempre cualquier respuesta establecida de vuelta (established, related, untracked) en la cadena `forward`.

> **Pista:** Usa listas de interfaces, o especifica las `in-interface` y `out-interface` para ser preciso al descartar (`drop`) o aceptar el tráfico. Puedes descartar puntualmente lo no deseado o descartarlo todo y aceptar solo hacia la WAN.

## Entrega
Entregable en PDF adjuntando:
- Capturas de la configuración de las interfaces y rutas creadas.
- Captura de las tres máquinas virtuales (Alumnos, Profesores, Admin) demostrando qué IPs han obtenido.
- Listado de reglas del firewall `/ip firewall filter export`.
- Demostración empírica de seguridad: un ping desde el alumno al profesor **debe fallar**, y un ping desde el alumno a Internet (8.8.8.8) **debe tener éxito**.
