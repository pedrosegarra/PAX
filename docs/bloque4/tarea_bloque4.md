# Tarea: Firewall básico y Redirección de Puertos

## Objetivo
Aplicar reglas de firewall restrictivas para proteger un router de borde y configurar el enrutamiento y NAT de forma que se pueda publicar un servidor web que reside en una zona desmilitarizada (DMZ).

## Descripción del Escenario

Dispones de un router (R1) conectado a tres redes diferenciadas mediante tres interfaces físicas:
1. **ether1 (WAN):** Conectada a Internet (obtiene IP dinámicamente o por estático, según indique el profesor).
2. **ether2 (LAN):** Red local interna, subred `192.168.10.0/24`.
3. **ether3 (DMZ):** Zona pública para servidores, subred `172.16.20.0/24`.

Dentro de la DMZ tienes un servidor web (HTTP, puerto 80) alojado en la IP `172.16.20.10`.

## Tareas a Realizar

### Fase 1. Enrutamiento y NAT
- Configura las direcciones IP correspondientes en ether2 y ether3.
- Asegúrate de que los dispositivos de la LAN puedan salir a Internet creando una regla de NAT (masquerade) que afecte solo al tráfico que sale por la interfaz WAN.

### Fase 2. Reglas de Protección (Input)
El router de borde debe estar protegido de accesos no autorizados:
- Crea una regla que permita el tráfico establecido y relacionado en la cadena `input`.
- Crea una lista de direcciones (Address List) llamada `Admin_IPs` y añade la IP del equipo del administrador de la LAN (ej. `192.168.10.50`).
- Crea una regla en `input` que permita todas las conexiones entrantes (como Winbox o SSH) pero **únicamente** para las direcciones de origen incluidas en la lista `Admin_IPs`.
- Crea una regla final (Drop) que descarte absolutamente todo el resto del tráfico destinado al router. ¡Atención! Asegúrate primero de que tu conexión no se va a cortar.

### Fase 3. Redirección de Puertos (Port Forwarding)
- Un usuario desde Internet debe poder acceder al servidor web alojado en la DMZ tecleando la IP pública del router.
- Configura una regla de destino NAT (`dst-nat`) en el Firewall que intercepte el tráfico entrante por el puerto 80 hacia la interfaz WAN y lo redirija a la IP interna `172.16.20.10`, puerto 80.

## Entrega
Se debe presentar un documento en PDF con capturas o un export en texto exportado desde RouterOS (`/ip firewall filter export` y `/ip firewall nat export`) donde se aprecien las reglas configuradas, junto a capturas demostrativas del funcionamiento:
- El administrador puede acceder por Winbox al router, pero un equipo distinto de la LAN no.
- El servidor de la DMZ responde a peticiones desde una máquina en la WAN.
