# Configuración de IP dinámica sobre interfaz física

## Índice

Introducción  
Ejemplo 1 — Configuración de IP dinámica sobre una interfaz física  
Eliminando la configuración aplicada  
Eliminación del cliente DHCP  

---

## Introducción

A lo largo de esta práctica trabajaremos sobre un escenario típico: la configuración de la interfaz que actuará como gateway, obteniendo su dirección IP de un servidor DHCP. Este enfoque es extremadamente habitual en entornos reales, donde el router no recibe una configuración fija, sino que depende de un proveedor externo (ISP, red corporativa, red del centro, laboratorio, etc.) para obtener sus parámetros de red.

• Primero, configuraremos direccionamiento IP dinámico, mediante un cliente DHCP sobre una interfaz física, el enfoque más simple y directo.  
• A continuación, configuraremos esa misma interfaz con direccionamiento IP estático, analizando qué cambia.  
• Después, daremos el salto conceptual al uso de bridges, aplicando una configuración de IP dinámica sobre un bridge, mediante la configuración de un cliente DHCP sobre el bridge.  
• Finalmente, cerraremos con la configuración de un bridge con IP estática, que es el modelo más habitual en entornos reales con múltiples routers configurados en una red LAN.  

Esta progresión permite observar cómo RouterOS separa claramente las capas de hardware, enlace y red, y prepara el terreno para configuraciones más avanzadas (LAN con múltiples puertos, firewall, etc.) que se abordarán más adelante en el curso.

---

## Ejemplo 1 — Configuración de IP dinámica sobre una interfaz física

Este escenario aborda una configuración usual en entornos reales: la obtención automática de parámetros de red mediante DHCP, configurando una interfaz física del router como cliente DHCP.

En este caso no se asigna ninguna dirección IP de forma manual. Será el servidor DHCP de la red quien proporcione al router:

• Dirección IP  
• Máscara de red  
• Puerta de enlace por defecto  
• (Opcionalmente) servidores DNS  

El objetivo de esta práctica es configurar la primera interfaz del router (ether1) como cliente DHCP, comprobar qué información recibe automáticamente y verificar que el router puede comunicarse a través de dicha interfaz sin intervención manual en el direccionamiento IP.

---

### Creación del cliente DHCP en la interfaz ether1

En primer lugar, vamos a comprobar que la interfaz ether1 no tiene ninguna IP configurada manualmente, tal como hemos visto en los tutoriales anteriores:

![img02](tema01/img10.png)
Una vez nos hemos asegurado de que no se ha aplicado configuración manual de IP sobre la interfaz ether1, podemos crear el cliente DHCP sobre dicha interfaz:
```sh
ip/dhcp-client/add interface=ether1 disable=no
```
---
![img02](tema01/img11.png)
En la captura, podemos revisar los clientes DHCP activos, tras ejecutar el comando:
```sh
ip/dhcp-client/print
```
En mi caso, el servidor DHCP de la red NAT ha asignado la IP 192.168.122.61/24, a la interfaz. Esta información la podemos obtener también desde el listado de IPs asignadas a interfaces, con el comando:
```sh 
ip/addresses/print
```
![img02](tema01/img12.png)

En la captura podemos observar el flag ‘D’, que indica que la asignación de IP se ha realizado de manera dinámica.

Si queremos obtener información detallada de la configuración asignada por DHCP, podemos ejecutar el siguiente comando:
```sh
ip/dhcp-client/print detail
```
![img02](tema01/img13.png)

En la captura podemos observar, entre otros:

• La IP asignada.
• La Puerta de enlace de la red  
• La IP del servidor DHCP  
• La IP del servidor DNS primario  
• …  

También podemos comprobar la configuración asignada por DHCP (IP, ruta por defecto y servidores DNS) ejecutando los siguientes comandos:
```sh
ip/address/print
```  
![img02](tema01/img14.png)
```sh
ip/route/print  
```
![img02](tema01/img15.png)
```sh
ip/dns/print
```  
![img02](tema01/img16.png)



---

Por último, Podemos probar la conectividad de red, ejecutando un ping a Google.com, por ejemplo, mediante el comando:

ping google.com

![img02](tema01/img17.png)

También podemos acceder al router utilizando un navegador web de la máquina anfitrión, a través de la IP configurada:

![img02](tema01/img18.png)


---

## Eliminando la configuración aplicada

Antes de continuar, vamos a eliminar toda la configuración aplicada al router, para volver a tener un dispositivo limpio, sin necesidad de aplicar la configuración de fábrica.

---

## Eliminación del cliente DHCP

Para mostrar los clientes DHCP configurados, ejecutamos el siguiente comando:
```sh
ip/dhcp-client/print
```

![img02](tema01/img19.png)
---

Para eliminar el cliente DHCP ejecutamos el siguiente comando:
```sh
ip/dhcp-client/remove <<índice>>
```
![img02](tema01/img20.png)

---

Al eliminar el cliente DHCP, se eliminará toda la configuración aplicada por el mismo. Podemos comprobarlo ejecutando los siguientes comandos:
```sh
ip/address/print  
```
![img02](tema01/img21.png)
```sh
ip/route/print  
```
![img02](tema01/img22.png)
```sh
ip/dns/print  
```

![img02](tema01/img23.png)


---

Ya tenemos nuestro router sin configuraciones por defecto, preparado para la siguiente práctica.


# Preguntas
### 1. ¿Qué es un cliente DHCP y para qué se utiliza?
<details>
<summary>Ver respuesta</summary>

Un cliente DHCP es un servicio que permite a un dispositivo obtener automáticamente su configuración de red (IP, máscara, gateway, DNS) desde un servidor DHCP.

</details>

### 2. ¿Qué parámetros de red puede proporcionar un servidor DHCP?
<details>
<summary>Ver respuesta</summary>

Dirección IP, máscara de red, puerta de enlace por defecto y servidores DNS.

</details>

### 3. ¿Por qué es habitual usar IP dinámica en la interfaz de gateway?
<details>
<summary>Ver respuesta</summary>

Porque en muchos entornos (como ISP o redes corporativas) la configuración es proporcionada automáticamente y puede cambiar, facilitando la gestión y evitando configuraciones manuales.

</details>

### 4. ¿Qué significa que una IP tenga el flag D (dynamic)?
<details>
<summary>Ver respuesta</summary>

Indica que la dirección IP ha sido asignada dinámicamente por un servidor DHCP.

</details>

### 5. ¿Qué diferencia hay entre IP dinámica e IP estática?
<details>
<summary>Ver respuesta</summary>

La IP dinámica se asigna automáticamente por DHCP, mientras que la IP estática se configura manualmente y no cambia.

</details>

### 6. ¿Qué función tiene la puerta de enlace por defecto?
<details>
<summary>Ver respuesta</summary>

Permite que el dispositivo se comunique con redes externas, como Internet.

</details>

### 7. ¿Qué papel juegan los servidores DNS?
<details>
<summary>Ver respuesta</summary>

Traducen nombres de dominio (como google.com) en direcciones IP.

</details>

### 8. ¿Qué comando permite añadir un cliente DHCP en una interfaz?
<details>
<summary>Ver respuesta</summary>

ip/dhcp-client/add interface=ether1 disable=no

</details>

### 9. ¿Qué comando muestra los clientes DHCP activos?
<details>
<summary>Ver respuesta</summary>

ip/dhcp-client/print

</details>

### 10. ¿Cómo se puede ver la IP asignada a una interfaz?
<details>
<summary>Ver respuesta</summary>

ip/address/print

</details>

### 11. ¿Qué comando muestra información detallada del cliente DHCP?
<details>
<summary>Ver respuesta</summary>

ip/dhcp-client/print detail

</details>

### 12. ¿Cómo comprobarías la tabla de rutas del router?
<details>
<summary>Ver respuesta</summary>

ip/route/print

</details>

### 13. ¿Qué comando permite comprobar los servidores DNS configurados?
<details>
<summary>Ver respuesta</summary>

ip/dns/print

</details>
### 14. ¿Qué información puedes obtener con ip/dhcp-client/print detail?
<details>
<summary>Ver respuesta</summary>

IP asignada, gateway, servidor DHCP, DNS, tiempo de concesión, entre otros.

</details>

### 15. ¿Cómo identificas una IP dinámica?
<details>
<summary>Ver respuesta</summary>

Por el flag "D" en el listado de direcciones IP.

</details>

### 16. ¿Qué indica la presencia de una ruta por defecto?
<details>
<summary>Ver respuesta</summary>

Que el router puede enviar tráfico hacia otras redes (normalmente Internet).

</details>

### 17. ¿Por qué es importante usar ping google.com?
<details>
<summary>Ver respuesta</summary>

Para verificar conectividad a Internet y resolución DNS.

</details>
### 18. ¿Qué ocurriría si no hay servidor DHCP?
<details>
<summary>Ver respuesta</summary>

El dispositivo no obtendría configuración IP automáticamente y no tendría conectividad.

</details>

### 19. ¿Por qué no se asigna manualmente una IP?
<details>
<summary>Ver respuesta</summary>

Porque se quiere automatizar la configuración y adaptarse a entornos dinámicos.

</details>

### 20. Explica cómo comprobar la conectividad
<details>
<summary>Ver respuesta</summary>

1. Ver IP: ip/address/print  
2. Ver ruta: ip/route/print  
3. Ver DNS: ip/dns/print  
4. Hacer ping a google.com  

</details>
### 21. ¿Qué comando elimina el cliente DHCP?
<details>
<summary>Ver respuesta</summary>

ip/dhcp-client/remove <indice>

</details>

### 22. ¿Qué ocurre al eliminarlo?
<details>
<summary>Ver respuesta</summary>

Se eliminan la IP, la ruta por defecto y los DNS asignados por DHCP.

</details>

### 23. ¿Cómo comprobar que se ha eliminado?
<details>
<summary>Ver respuesta</summary>

Revisando:

ip/address/print  
ip/route/print  
ip/dns/print  

</details>