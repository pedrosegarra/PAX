# Configuración de una DMZ.

# Índice

Introducción  
Ejemplo de configuración de DMZ con Mikrotik 
Escenario de partida  
Creación de los bridges DMZ y LAN 
Aplicando la configuración de red del puerto ether1 con un cliente DHCP  
Aplicando la configuración de red de los bridges LAN y DMZ de manera estática  
Creando los servidores DHCP para bridge-lan y bridge-dmz  
Configurando NAT, para proporcionar salida a internet a las redes DMZ y LAN  
Configurando las reglas de firewall necesarias para proteger nuestra infraestructura  
Reglas INPUT: proteger el router  
Reglas FORWARD: políticas entre zonas 
Publicación del servicio HTTP a internet (WAN → DMZ)  
Corregir la ordenación de las reglas aplicadas  
Configuración del servidor http en Alpine Linux y validación del escenario 

---

# Introducción

En entornos reales de administración de redes, no todos los sistemas deben tener el mismo nivel de exposición ni el mismo grado de confianza. Los servicios que deben ser accesibles desde Internet (servidores web, correo, aplicaciones, VPN, etc.) suponen un riesgo inherente, ya que amplían la superficie de ataque de la infraestructura. La DMZ (Demilitarized Zone) surge precisamente como un mecanismo para aislar estos servicios del resto de la red interna, reduciendo el impacto de una posible intrusión.

Una DMZ es una red intermedia, separada tanto de Internet como de la LAN interna, en la que se ubican los servidores expuestos públicamente. La idea fundamental es clara: si un servicio publicado es comprometido, el atacante no debe poder acceder directamente a la red interna, sino quedar contenido dentro de la zona desmilitarizada.

En MikroTik, la implementación de una DMZ no es un modo automático ni una casilla que se marque, sino el resultado de una arquitectura de red bien definida y de un conjunto coherente de reglas de firewall y NAT. Esto encaja perfectamente con la filosofía del sistema: control explícito, reglas claras y políticas de seguridad basadas en el principio de mínimo privilegio.
