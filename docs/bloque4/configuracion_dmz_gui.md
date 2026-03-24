# Configuración de una DMZ desde GUI.


# Índice

## Introducción
### Ejemplo de configuración de DMZ con Mikrotik.
### Escenario de partida
#### Creación de los bridges DMZ y LAN
#### Aplicando la configuración de red del puerto ether1 con un cliente DHCP
#### Conectando al panel web del router.
#### Aplicando la configuración de red de los bridges LAN y DMZ de manera estática
#### Creando los servidores DHCP para bridge-lan y bridge-dmz
#### Configurando NAT, para proporcionar salida a internet a las redes DMZ y LAN.
#### Configurando las reglas de firewall necesarias para proteger nuestra infraestructura.
##### Reglas INPUT: proteger el router
##### Reglas FORWARD: políticas entre zonas
##### Publicación del servicio HTTP a internet (WAN → DMZ)
##### Corregir la ordenación de las reglas aplicadas.
#### Configuración del servidor http en Alpine Linux y validación del escenario.
### Añadiendo un nuevo segmento de red.
#### Reserva de IP (Lease) estática para R2 en R1
#### Configuración de cliente DHCP en ether1 de R2
#### Configuración de la red 192.168.1.0/24 en R2
#### Configuración del servidor DHCP para la red de oficina, en R2
#### Configuración Ruta de R1 a la red de oficina
### Conclusión.

# Introducción

En esta práctica vamos a repetir íntegramente el ejercicio de configuración de una DMZ, manteniendo el mismo escenario, la misma arquitectura de red y las mismas políticas de seguridad ya trabajadas.

La diferencia fundamental será el enfoque: en lugar de realizar la configuración desde la consola mediante comandos, utilizaremos exclusivamente los entornos gráficos de RouterOS, concretamente WinBox y el panel web (WebFig).

El objetivo es analizar cómo se traduce cada instrucción de terminal en su equivalente visual, comprender la estructura interna de RouterOS desde otra perspectiva y observar las diferencias.

De este modo, podremos comparar ambos métodos de trabajo —CLI y GUI— evaluando ventajas, limitaciones y escenarios de uso recomendados en entornos profesionales reales.

# Ejemplo de configuración de DMZ con Mikrotik.

En este escenario vamos a configurar un router MikroTik que actúa como punto central de interconexión y control de seguridad entre tres zonas claramente diferenciadas, cada una con un nivel de confianza distinto:

- **WAN (Wide Area Network)**: la zona WAN representa la conexión a Internet, es decir, una red no confiable sobre la que no tenemos control. La WAN se considerará la zona de menor confianza.
- **DMZ (Zona Desmilitarizada)**: la zona DMZ representa una red intermedia diseñada para alojar servicios que deben ser accesibles desde Internet, pero que no deben comprometer la red interna. La DMZ actúa como una zona tampón: si un servidor es comprometido, el daño queda contenido.
- **LAN (Red interna)**: la zona LAN representa la red interna de confianza, donde se encuentran los equipos de trabajo y los sistemas no expuestos públicamente. La LAN es la zona de mayor confianza y debe estar estrictamente protegida.

![img02](img/80.png)

Para ello, vamos a crear un nuevo escenario en GNS3, que iremos configurando paso a paso.

![img02](img/81.png)

# Escenario de partida

Vamos a partir del siguiente escenario en GNS3, que iremos configurando paso a paso, hasta llegar al objetivo marcado en el punto anterior.

![img02](img/82.png)

Vamos a conectar el nodo NAT con el puerto ethernet0 del router (que se corresponderá con la interfaz ether1 en la consola de Mikrotik). También añadiremos un cliente web y un cliente WinBox a la red NAT, para poder ejecutar los primeros pasos de configuración del router.

![img02](img/83.png)

Iniciaremos el escenario, pulsando el botón Play, y abriremos WinBox, para poder conectar al router.

![img02](img/84.png)

Antes de continuar, vamos a validar que partimos de un router en blanco, consultando los siguientes paneles:

Accedemos al panel Interfaces → Interface, donde debemos ver únicamente las 4 interfaces ether, y la interfaz IO:

![img02](img/85.png)

Accedemos al panel IP → Addresses, donde no debe existir ningún registro:

![img02](img/86.png)

Accedemos al panel IP → Routes, donde no debe existir ningún registro:

![img02](img/87.png)

Accedemos al panel IP → Firewall → NAT, donde no debe existir ningún registro:

![img02](img/88.png)

Accedemos al panel IP → Firewall → Filter Rules, donde no debe existir ningún registro:

![img02](img/89.png)

Accedemos al panel IP → DHCP Client, donde no debe existir ningún registro:

![img02](img/90.png)

Accedemos al panel IP → DHCP Server, donde no debe existir ningún registro:

![img02](img/91.png)

# Creación de los bridges DMZ y LAN

Para empezar, vamos a crear los bridges DMZ y LAN, sobre los que aplicaremos la configuración de las redes internas. Para ello, accedemos al panel Bridge → Bridge, y pulsamos el botón “New”.

![img02](img/92.png)

Rellenamos los datos para el bridge de la red LAN, como se muestra en la captura, y pulsamos el botón “Ok”.

![img02](img/93.png)

Repetimos la operación para el bridge de la red DMZ.

![img02](img/94.png)

Realizamos una comprobación rápida, antes de continuar. En el listado de bridges, deben aparecer los dos nuevos elementos creados.

![img02](img/95.png)

Si todo es correcto, pasamos a añadir los puertos físicos a cada bridge. En este caso, añadiremos ether2 y ether3 al bridge LAN, y ether4 al bridge DMZ.

Para ello, navegamos a la opción Bridge → Ports, y pulsamos el botón “New”.

![img02](img/96.png)

El formulario nos permite asociar una interface con un bridge. En este caso, asociamos la interfaz ether2 al bridge LAN. Para guardar la asociación, pulsamos el botón “OK”.

![img02](img/97.png)

Repetimos el proceso para la interfaz ether3, asociándola al bridge LAN:

![img02](img/98.png)

Y repetimos el proceso para configurar la interfaz ether4, asociándola al bridge DMZ.

![img02](img/99.png)

Comprobamos que las interfaces se han añadido correctamente a los bridges correspondientes:

![img02](img/100.png)

# Aplicando la configuración de red del puerto ether1 con un cliente DHCP

Para continuar, vamos a configurar la dirección IP de la interfaz ether1, creando un cliente DHCP, que obtenga la dirección IP del servidor DHCP al que esté conectada la red.

Para ello, accedemos al panel IP → DHCP Client, y pulsamos el botón New.

![img02](img/101.png)

En el formulario, seleccionamos la interfaz a la que asignaremos el cliente, configuramos las opciones que muestra la siguiente imagen, y pulsamos el botón “OK”.

![img02](img/102.png)

Donde:

- Enabled activa inmediatamente el cliente.
- use-peer-dns indica al router que use los servidores DNS que entregue el proveedor.
- use-peer-ntp sincroniza hora si el ISP lo ofrece. Útil para logs y firewall.
- add-default-route crea automáticamente la ruta por defecto (0.0.0.0/0). Sin esto, no hay salida a Internet.

Esperamos unos segundos, y comprobamos que el cliente DHCP se ha creado correctamente, recibiendo una IP del servidor DHCP de la red NAT.

![img02](img/103.png)

También podemos ver la IP asignada en el panel IP → Addresses.

![img02](img/104.png)
En el panel IP → Routes podemos ver la configuración de la ruta por defecto, y la ruta de acceso a la red NAT.

![img02](img/105.png)

En el panel IP → DNS podemos observar los servidores DNS asignados dinámicamente por el servidor DHCP.

![img02](img/106.png)

Podemos comprobar la conectividad con internet, por ejemplo, utilizando la herramienta tools -> traceroute.

![img02](img/107.png)

# Conectando al panel web del router.

Antes de continuar, vamos a conectar al panel web del router, para continuar la configuración desde el entorno gráfico que proporciona (que mejora sensiblemente la usabilidad con respecto al appliance de WinBox en GNS3).

Para ello, abrimos un cliente conectado a la red Wan, y accedemos a la IP asignada por el servidor DHCP a la interfaz ether1 (en mi caso, 192.168.122.72).

![img02](img/108.png)

Tras realizar login, pulsamos sobre la opción “Advanced” para mostrar todas las opciones de la interfaz gráfica.

![img02](img/109.png)

# Aplicando la configuración de red de los bridges LAN y DMZ de manera estática

Vamos a configurar la dirección IP de los bridges bridge-lan y bridge-dmz, para que su direccionamiento IP se corresponda con los datos del diagrama inicial.

En primer lugar, configuramos la dirección IP de bridge-lan, que se corresponderá con la puerta de enlace de la red 192.168.0.0/24.

Para ello, accedemos al panel IP → Addresses y creamos un nuevo registro, pulsando la opción “New”.

![img02](img/110.png)

Rellenamos el formulario con los datos necesarios para configurar la IP del bridge-lan, y pulsamos el botón “Ok”.

![img02](img/111.png)

Repetimos la operación para configurar la dirección IP de bridge-dmz. La IP se corresponderá con la puerta de enlace de la red 10.10.0.0/24.

![img02](img/113.png)

Comprobamos que todo ha ido bien, consultando el panel IP → Addresses.

![img02](img/114.png)

# Creando los servidores DHCP para bridge-lan y bridge-dmz

Vamos a crear dos servidores DHCP. Uno para bridge-lan, que se encargará de proporcionar parámetros de red a los clientes conectados, y uno para bridge-dmz, de manera que podamos centralizar la configuración de red de nuestros servidores.

Para ello, utilizaremos en los dos casos el panel IP → DHCP Server, pulsando el botón DHCP Setup.

![img02](img/115.png)

Para el servidor DHCP de bridge-lan, aplicaremos los parámetros mostrados en las siguientes capturas:

![img02](img/116.png)
![img02](img/117.png)
![img02](img/118.png)
![img02](img/119.png)
![img02](img/120.png)
![img02](img/121.png)
![img02](img/122.png)
Para el servidor DHCP de bridge-dmz, volvemos a lanzar el asistente, y aplicaremos los parámetros mostrados en las siguientes capturas:

![img02](img/123.png)
![img02](img/124.png)
![img02](img/125.png)
![img02](img/126.png)
![img02](img/127.png)
![img02](img/128.png)
![img02](img/129.png)
Podemos comprobar los datos de los servidores DHCP creados, en el panel IP → DHCP Servers:

![img02](img/130.png)

Si observamos el listado de servidores DHCP, podemos observar que el asistente ha asignado un nombre no demasiado identificativo a los servidores DHCP creados, y a los DHCP pool asociados. Para modificar el nombre del servidor, hacemos click sobre el nombre, y se abrirá el formulario de edición de los parámetros del servidor DHCP. Vamos a renombrar el servidor DHCP de la red lan a dhcp-lan, y pulsamos el botón “Ok”.

![img02](img/131.png)

Realizamos la misma operación, renombrando el servidor DHCP de la DMZ a dhcp-dmz.

![img02](img/132.png)

Podemos realizar la misma operación con los pool de direcciones IP, en el panel IP→pool.

![img02](img/133.png)

A continuación, vamos a asignar un lease estático para nuestro servidor DMZ. Le asignaremos la IP 10.10.0.10, teniendo en cuenta que su dirección MAC es 02:42:b1:8b:51:00.

Para ello, accedemos a la pestaña leases del panel IP → DHCP Server y pulsamos el botón “New”.

![img02](img/134.png)

Rellenamos los datos del formulario, y pulsamos el botón “Ok”.

![img02](img/135.png)

Podemos comprobar que todo ha ido bien, observando la pestaña leases del panel IP → DHCP Server.

![img02](img/136.png)

Para Validar que todo ha ido bien, podemos conectar al router el cliente (utilizando el puerto ethernet1 del router en GNS3) y el servidor (utilizando el puerto ethernet3 del router en GNS3).

![img02](img/137.png)

![img02](img/138.png)

Nos aseguramos de que al iniciar solicitaran dirección IP al servidor DNS, editando la configuración de ambos elementos, pulsando botón derecho sobre cada uno de ellos, y seleccionando la opción “Edit config”.

![img02](img/139.png)

Dentro del fichero de configuración, debemos eliminar los comentarios de las líneas que fuerzan la autoconfiguración de red mediante DHCP:

![img02](img/140.png)

Una vez editada la configuración, guardamos los cambios, y arrancamos la instancia. Repetimos el proceso en el otro appliance.

Una vez arrancadas las dos instancias, podemos volver a la configuración del router, y observar los leases asignados, en la pestaña Leases del panel IP→DHCP Server:

![img02](img/141.png)

Podemos observar cómo existen dos leases activos.

- El lease estático, configurado para asignar IP al servidor de la DMZ.
- Un lease dinámico, asignado al cliente de la red LAN, y otro asignado al appliance de Winbox, utilizado para configurar el router, si está configurado para obtener IP por DHCP.

![img02](img/142.png)

# Configurando NAT, para proporcionar salida a internet a las redes DMZ y LAN.

En nuestro escenario, los equipos de la LAN y los servidores de la DMZ utilizan direcciones IP privadas, que no son enrutables directamente en Internet. Para que estos dispositivos puedan comunicarse con redes externas, el router debe traducir sus direcciones privadas a la dirección IP pública (o de salida) de la interfaz WAN.

La técnica que utilizaremos es src-nat con masquerade, que es el método más habitual cuando la interfaz WAN obtiene su dirección IP de forma dinámica (por ejemplo, mediante DHCP). De esta manera, todo el tráfico que salga de la LAN o la DMZ hacia Internet será identificado como procedente del router, y las respuestas podrán volver correctamente a su destino original.

Aunque en MikroTik sería posible utilizar una única regla NAT genérica para dar salida a Internet a todas las redes internas, en este escenario es mucho más recomendable crear una regla NAT diferenciada para cada bridge (LAN y DMZ).

Crear una regla NAT para cada bridge permite:

- Leer la configuración de un vistazo (qué red sale a Internet).
- Controlar cada red de forma independiente (activar, desactivar o depurar solo LAN o solo DMZ).
- Mantener coherencia con el diseño de seguridad: no todas las redes internas son iguales.
- Facilitar el mantenimiento y la docencia.

Para crear la regla NAT para el tráfico de la red LAN, accedemos a la pestaña NAT del panel IP→Firewall y pulsamos el botón “New”.

![img02](img/143.png)

Rellenamos el formulario con los siguientes parámetros:

![img02](img/144.png)

Para crear la regla NAT para el tráfico de la red DMZ, realizamos el mismo procedimiento.

![img02](img/145.png)

Comprobamos que las reglas se han creado correctamente en la pestaña NAT del panel IP→Firewall.

![img02](img/146.png)

Una vez comprobada la configuración, podemos tratar de conectar a internet con el cliente o con el servidor:

![img02](img/148.png)

# Configurando las reglas de firewall necesarias para proteger nuestra infraestructura.

En esta sección abordaremos la configuración del firewall del router MikroTik, uno de los elementos clave para garantizar la seguridad de nuestra infraestructura de red.

Para mantener una configuración clara y coherente, separaremos las reglas según su función, trabajando de forma diferenciada sobre las cadenas principales:

- Input: reglas que controlan el tráfico dirigido al propio router (gestión, servicios y diagnóstico).
- Forward: reglas que regulan el tráfico que atraviesa el router, es decir, las comunicaciones entre LAN, DMZ y WAN.

Esta separación nos permitirá:

- aplicar el principio de mínimo privilegio,
- entender fácilmente qué tráfico se permite y cuál se bloquea,
- y evitar configuraciones confusas o peligrosas.

# Reglas INPUT: proteger el router

Las reglas input del firewall controlan el tráfico que va dirigido al propio router, no a los equipos de la red. Es decir, deciden quién y cómo puede comunicarse con el router para tareas como administración, diagnóstico o servicios internos.

Aceptar conexiones ya establecidas / relacionadas

Vamos a añadir una regla que permita al router aceptar tráfico que forma parte de conexiones ya establecidas o que está relacionado con una conexión previa válida.

Para ello, accedemos a la pestaña Filter Rules del panel IP→Firewall, y pulsamos el botón “New”.

![img02](img/149.png)

Y rellenamos el formulario, para crear la regla.

![img02](img/150.png)

Esta regla simplifica la configuración, permitiendo tráfico legítimo ya asociado a comunicaciones válidas.

## Bloquear conexiones inválidas

Vamos a añadir una regla que descarte paquetes marcados como invalid, es decir, tráfico que no pertenece a ninguna conexión válida o que está mal formado.

Pulsamos el botón “New”, y rellenamos el formulario con los siguientes parámetros.

Permitir ICMP (ping/troubleshoting)

También puede ser interesante permitir el tráfico ICMP dirigido al router, necesario para funciones básicas de diagnóstico y control. Creamos una regla con los siguientes parámetros:

![img02](img/151.png)

## Permitir ICMP (pint/troubleshoting)

También puede ser interesante permitir el tráfico ICMP dirigido al router, necesario
para funciones básicas de diagnóstico y control. Creamos una regla con los
siguientes parámetros:

![img02](img/152.png)

## Permitir administración SOLO desde LAN

Con la siguiente regla, aceptaremos conexiones que permitan el acceso de administración al router únicamente desde la red LAN, usando los servicios de SSH (22), WinBox (8291) o http(80).

![img02](img/153.png)

## Bloqueo final de todo lo demás hacia el router

Para finalizar, bloquearemos todo el tráfico dirigido al router que no haya sido permitido explícitamente antes.

![img02](img/154.png)

Es la regla que cierra la cadena input y garantiza que el router no exponga servicios innecesarios. Sin esta regla, cualquier tráfico no contemplado quedaría permitido, lo que supone un riesgo claro de seguridad.

A partir de este momento se bloquearán todas las conexiones nuevas desde la red WAN al router, para su administración.

## Comprobación de reglas input aplicadas.

Para comprobar las reglas aplicadas hasta el momento, accedemos a la pestaña Filter Rules del panel ip→firewall:

![img02](img/155.png)

Podemos observar todas las reglas de filtrado configuradas hasta el momento.

![img02](img/156.png)
Cabe recordar que las reglas de filtrado se ejecutan en orden, de arriba abajo, hasta encontrar una regla cuyas condiciones coinciden con los datos del paquete analizado, ejecutando la regla sin analizar las siguientes.

# Reglas FORWARD: políticas entre zonas

Las reglas forward del firewall controlan el tráfico que atraviesa el router, es decir, las comunicaciones entre redes distintas como LAN, DMZ y WAN.

Estas reglas deciden:

- Qué redes pueden comunicarse entre sí,
- En qué dirección,
- Bajo qué condiciones.

## Permitir established/related

En primer lugar, debemos añadir una regla que permita el paso del tráfico que pertenece a conexiones ya establecidas o que está relacionado con una conexión válida, pero en este caso atravesando el router entre redes.

![img02](img/157.png)

## Bloquear conexiones inválidas

De la misma manera, deberemos añadir una regla que descarte el tráfico marcado como invalid que intenta atravesar el router entre distintas redes.

![img02](img/158.png)

## Permitir tráfico de red LAN hacia WAN

Para permitir que los equipos de la red LAN puedan salir a Internet, añadiremos una regla, autorizando el tráfico que entra al router desde bridge-lan y sale por la interfaz WAN (ether1).

![img02](img/159.png)

## Permitir tráfico de red LAN hacia DMZ

Para permitir que los equipos de la red LAN puedan conectar a los servicios desplegados en la DMZ, añadiremos una regla, autorizando el tráfico que entra al router desde bridge-lan y sale por el bridge-dmz.

![img02](img/160.png)

## Permitir tráfico de red DMZ hacia WAN

Si necesitamos que los equipos de la DMZ dispongan de acceso a internet, deberemos añadir una regla, autorizando el tráfico que entra al router desde bridge-dmz y sale por la interfaz WAN (ether1).

![img02](img/161.png)

![img02](img/162.png)

## Bloquear tráfico de red DMZ hacia LAN

Para proteger la red LAN de accesos no autorizados, deberemos bloquear cualquier intento de comunicación iniciado desde la DMZ hacia la LAN, evitando que los sistemas expuestos en la red perimetral puedan acceder a la red interna.

![img02](img/163.png)

## Bloquear tráfico de red WAN hacia LAN

De la misma manera, deberemos crear una regla que bloquee cualquier intento de conexión iniciado desde la WAN hacia la LAN, impidiendo accesos directos desde Internet a la red interna.

![img02](img/164.png)

## Bloquear tráfico de red WAN hacia DMZ

Dado que solo será necesario exponer los servicios necesarios para su acceso desde internet, deberemos bloquear cualquier conexión iniciada desde la WAN hacia la DMZ, dejando la red perimetral cerrada por defecto.

![img02](img/165.png)

## Bloquear cualquier otro tipo de tráfico

Por último, bloquearemos todo el tráfico que atraviesa el router y que no haya sido permitido explícitamente por reglas anteriores.

![img02](img/166.png)

## Comprobación de reglas forward aplicadas.

Para comprobar las reglas aplicadas hasta el momento, accedemos a la pestaña Filter Rules del panel ip→firewall. Si lo queremos, podemos filtrar las reglas, para mostrar solo las que contengan la cadena forward:

![img02](img/167.png)

# Publicación del servicio HTTP a internet (WAN → DMZ)

En esta sección vamos a publicar un servicio HTTP alojado en un servidor de la DMZ, permitiendo su acceso desde Internet de forma controlada y segura.

La publicación de servicios no consiste en “abrir la DMZ”, sino en crear excepciones muy concretas al firewall.

## DST-NAT: redirigir TCP/80 hacia el servidor DMZ

Para redirigir las peticiones HTTP (TCP/80) que llegan desde Internet a la interfaz WAN (ether1) hacia el servidor web ubicado en la DMZ (10.10.0.10), en primer lugar, debemos crear una regla NAT, que redirija la petición.

Para ello, accedemos a la pestaña NAT del panel IP→firewall y pulsamos el botón “New”.

![img02](img/168.png)

Y rellenamos los datos del formulario, para configurar la redirección, como sigue:

![img02](img/169.png)

![img02](img/170.png)

Esta regla no abre el acceso por sí sola: únicamente realiza la redirección. Para que el tráfico funcione de forma segura, debe ir acompañada de una regla forward específica que permita ese tráfico y mantenga bloqueado todo lo demás.

## FILTER: permitir el forward WAN → DMZ solo para HTTP

A continuación, añadiremos la regla que permitirá el tráfico HTTP iniciado desde Internet hacia el servidor web de la DMZ (10.10.0.10), autorizando únicamente nuevas conexiones TCP al puerto 80.

Para ello, accedemos a la pestaña Filter Rules del panel IP→firewall y pulsamos el botón “New”.

![img02](img/171.png)

Y rellenamos los datos del formulario, para configurar la regla, como sigue:

![img02](img/172.png)
# Corregir la ordenación de las reglas aplicadas.

Para validar la configuración del router, vamos a revisar todas las reglas forward, ejecutando:

![img02](img/173.png)

Si observamos la salida, la regla 12 bloquea todas las conexiones dirigidas desde la red WAN a la red DMZ, y la regla 14 habilita el protocolo HTTP desde la red WAN a la red DMZ.

Dado que las reglas se evaluarán en orden, y se ejecutará únicamente la primera regla que coincida con el estado de la conexión, la regla 14 nunca llegará a evaluarse, dado que siempre se bloqueará el tráfico que la cumple por la regla 12.

![img02](img/174.png)

Para modificar el orden de evaluación, podemos arrastrar la regla 14 encima de la regla 12.

Con este cambio, nuestras reglas serán totalmente funcionales, y podemos validar el escenario completo.

# Configuración del servidor http en Alpine Linux y validación del escenario.

Para instalar e iniciar el servicio http en Alpine Linux, debemos abrir un terminal, y ejecutar el siguiente script:

```sh
apk update;
apk add python3;
mkdir -p /var/www;
echo "HTTP OK desde Alpine (DMZ)" > /var/www/index.html;
cd /var/www;
python3 -m http.server 80;
```

Si todo ha funcionado correctamente, tendremos validada la conexión entre DMZ y WAN.

Abrimos el navegador web conectado en red LAN, y tratamos de acceder a la ip del servidor http (10.10.0.10). Si todo ha ido bien, habremos validado la conexión entre la red LAN y la red DMZ.

![img02](img/175.png)

A continuación, podemos conectar un cliente con navegador web en la red LAN.

![img02](img/176.png)

Si abrimos el cliente, y tratamos de acceder al puerto 80 del router, y todo está bien configurado, nos devolverá el contenido de la página web servida desde el servidor de la DMZ.

![img02](img/177.png)

Por último, vamos a tratar de conectar desde el servidor en la DMZ al cliente, a través de su IP privada. En la captura podemos comprobar como no se obtiene respuesta de la petición:

![img02](img/178.png)

Con esta configuración dejamos el servicio HTTP publicado de forma controlada y segura, validando en la práctica cómo NAT y firewall trabajan conjuntamente sin comprometer el aislamiento entre LAN, DMZ y WAN.

# Añadiendo un nuevo segmento de red.

Nuestra red LAN ha crecido en tamaño y complejidad, por lo que hemos decidido segmentar la red de oficina situándola detrás de un segundo router dentro de la propia LAN, siguiendo el esquema propuesto.

![img02](img/179.png)

Para esta nueva red de oficina utilizaremos el rango 192.168.1.0/24, que quedará diferenciada de la LAN principal y gestionada por el router interno.

# Reserva de IP (Lease) estática para R2 en R1

Antes de configurar el segundo router (en adelante R2), es recomendable asegurar que su interfaz conectada a la LAN principal reciba siempre la misma dirección IP.

Aunque podríamos configurar una IP estática directamente, en este escenario optaremos por una reserva DHCP en el router principal (en adelante R1).

En primer lugar, vamos a obtener la dirección MAC de ether1 en R2, mirando la configuración del appliance.

![img02](img/180.png)

Copiamos la dirección MAC, y configuramos un lease en el servidor DHCP de la LAN, en R1, accediendo a la pestaña Leases del panel IP→DHCP Server.

![img02](img/181.png)

# Configuración de cliente DHCP en ether1 de R2

Una vez creada la reserva DHCP en el router principal (R1), el siguiente paso consiste en iniciar el segundo router (R2) y configurar su interfaz ether1 como cliente DHCP.

Para ello, conectamos WinBox al puerto ether1, conectamos al router, y creamos el nuevo cliente DHCP, en el panel IP→DHCP Client.

![img02](img/182.png)

Desconectamos WinBox del router, conectamos el primer puerto del router al switch, para que el cliente DHCP pueda obtener IP, y podremos acceder a la configuración del router desde un navegador conectado a la red LAN de R1, utilizando la IP reservada.

![img02](img/183.png)

# Configuración de la red 192.168.1.0/24 en R2

Una vez que el router interno (R2) dispone de conectividad con la LAN principal, el siguiente paso consiste en configurar la nueva red de oficina, que quedará segmentada detrás de este segundo router.

Esta red utilizará el rango 192.168.1.0/24 y R2 actuará como puerta de enlace de esta nueva subred, gestionando su direccionamiento y encaminando el tráfico hacia la LAN principal a través de R1.

Para el ejemplo, vamos a añadir las tres interfaces restantes de R2 al bridge “bridge-oficina”. Para ello, accedemos a la pestaña Bridge del panel Bridge, pulsamos el botón “New” y rellenamos el formulario:

![img02](img/184.png)

Para añadir los puertos al bridge, vamos a la pestaña Ports del mismo panel, pulsamos el botón “New” y rellenamos el formulario para añadir ether2 al bridge-oficina.

![img02](img/185.png)

Repetimos el proceso para ether3 y ether4, quedando así:

![img02](img/186.png)

Una vez configurado el bridge, y añadidos los puertos al mismo, debemos asignarle IP, en el panel IP→Addresses, pulsando el botón “New” y rellenando el formulario.

![img02](img/187.png)

# Configuración del servidor DHCP para la red de oficina, en R2

Para facilitar la gestión de la red de oficina, vamos a configurar un servidor DHCP, que gestione la asignación de IPs a los elementos conectados al bridge-oficina.

Para ello, accedemos al panel IP→DHCP Server, y pulsamos el botón DHCP Setup, asignando los siguientes valores:

![img02](img/188.png)

![img02](img/189.png)

Si queremos, podemos modificar el nombre del servidor DHCP y del pool de direcciones IP, siguiendo los pasos del ejemplo realizado anteriormente.

Una vez finalizado el proceso, podemos observar como los clientes reciben IP, en la pestaña Leases del panel IP→DHCP Server.

![img02](img/190.png)

Si abrimos un cliente de la red de oficina, este podrá conectar al panel web del router, a través de un navegador, accediendo a http://192.168.1.1

![img02](img/191.png)

Pero no podrá acceder al panel de control del router R1, a través de la IP 192.168.0.1.

![img02](img/192.png)

Esto ocurre porque todavía no hemos configurado el enrutamiento entre R1 y la nueva red de oficina situada detrás de R2. Aunque los equipos de la red de oficina pueden enviar tráfico hacia R1 (gracias a que su puerta de enlace es R2, y R2 sí conoce a R1), R1 no tiene ninguna ruta hacia 192.168.1.0/24, por lo que cuando recibe las respuestas destinadas a esa subred no sabe a qué siguiente salto reenviarlas.

El resultado es un fallo de conectividad por ausencia de ruta de retorno: el tráfico “sale”, pero la respuesta no puede volver al origen. La solución consiste en añadir en R1 una ruta estática hacia 192.168.1.0/24 indicando como gateway la IP de R2 en la LAN principal (por ejemplo, 192.168.0.2).

# Configuración Ruta de R1 a la red de oficina

Para resolver el problema de enrutamiento entre R1 y la red de oficina situada detrás de R2, es necesario acceder a la configuración del router principal (R1) y dirigirse al apartado IP → Routes.

En este punto añadiremos una nueva ruta estática, indicando que la red 192.168.1.0/24 es alcanzable a través del router interno (R2), cuya dirección en la LAN principal actúa como siguiente salto.

Para ello, pulsamos el botón “New” y completamos el formulario.

![img02](img/193.png)

Podemos comprobar la ruta configurada en el panel de rutas.

![img02](img/194.png)

Y podemos validar el acceso, desde el cliente de la red de oficina.


![img02](img/195.png)

También podemos validar el acceso a internet, dado que no se requiere configurar un NAT adicional (con la configuración de NAT en R1 es suficiente.

![img02](img/196.png)

Para finalizar, podemos reforzar la arquitectura configurando en R2 las reglas de seguridad que consideremos oportunas en su firewall, siguiendo el mismo criterio aplicado previamente en R1. De este modo, cada docente podrá ajustar el nivel de segmentación y control de acceso según los objetivos didácticos del ejercicio, ya sea manteniendo una configuración básica de enrutamiento o incorporando políticas más restrictivas que refuercen la separación entre redes.

# Conclusión.

A lo largo de este manual hemos diseñado y desplegado una arquitectura completa basada en RouterOS, partiendo de un escenario simple y evolucionándolo progresivamente hacia una infraestructura segmentada y estructurada. Hemos trabajado la separación de zonas (WAN, DMZ y LAN), la publicación controlada de servicios mediante NAT, la aplicación rigurosa de políticas de firewall siguiendo el principio de mínimo privilegio y, finalmente, la ampliación del entorno mediante un segundo router interno con enrutamiento estático.

El objetivo no ha sido únicamente lograr conectividad, sino comprender el porqué de cada decisión técnica: cómo fluye el tráfico, cómo se establece el retorno, cómo interactúan NAT y firewall, y cómo un diseño coherente reduce riesgos y facilita el mantenimiento. Esta visión permite pasar de una configuración “funcional” a una arquitectura razonada y profesional.

El escenario desarrollado constituye una base sólida sobre la que cada docente puede profundizar según el nivel del grupo: endurecimiento adicional del firewall, segmentación más granular, control por listas de direcciones, implementación de protocolos dinámicos de enrutamiento o simulación de entornos corporativos más complejos.

Con ello cerramos un bloque formativo completo que combina diseño, implementación y validación práctica, proporcionando al alumnado una experiencia cercana a situaciones reales de administración de redes.
