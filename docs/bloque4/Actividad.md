# Actividad a entregar
Partiendo de la infraestructura ya configurada y operativa, resultado de haber seguido el tutorial “Configuración de una dmz”, explica cómo añadirías las reglas necesarias para permitir que la aplicación web ubicada en la DMZ pueda conectarse al servidor MariaDB de la LAN, manteniendo el resto del tráfico bloqueado por seguridad.

Puedes utilizar los siguientes datos:

IP de la aplicación web en DMZ: 10.10.0.10
IP del servidor MariaDB en LAN: 192.168.0.50
Puerto de MariaDB: 3306/tcp
Explica brevemente:

En qué cadena del firewall (input o forward) se debe crear la regla.
Qué tráfico se debe permitir, especificando:
origen,
destino,
protocolo y puerto.