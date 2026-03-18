# Configuración de IP estática sobre interfaz física

## Índice

- Introducción
- Ejemplo 2 — Configuración de IP estática sobre una interfaz física
- Asignación manual de IP
- Asignación manual de ruta por defecto
- Asignación manual de servidores DNS
- Eliminación manual de la configuración aplicada

## Introducción

A lo largo de esta práctica trabajaremos sobre un escenario típico: la configuración de la interfaz que actuará como gateway, asignando su dirección IP estática de manera manual. Este enfoque es extremadamente habitual en entornos reales, donde el router no recibe una configuración dinámica, sino que depende de la configuración manual del operador de red.

## Ejemplo 2 — Configuración de IP estática sobre una interfaz física

Este segundo escenario representa la configuración manual más simple y directa posible en RouterOS: asignar manualmente una dirección IP a una interfaz física del router para que la interfaz pueda interactuar con la red existente.

Para ello, vamos a configurar la primera interfaz del router con la misma IP que tenía asignada por DHCP (en mi caso 192.168.122.61/24), para posteriormente comprobar que podemos acceder al router a través de esta interfaz.

## Asignación manual de IP

En primer lugar, vamos a listar las interfaces físicas de nuestro router, para seleccionar cuál configuraremos, ejecutando el siguiente comando:

```
interface/print
```
![img02](tema01/img30.png)
En mi caso, configuraré la interfaz ether1, asignándole la IP 192.168.122.61/24, ejecutando:

```
ip/address/add address=192.168.122.61/24 interface=ether1 comment="Gateway – IP estatica"
```
![img02](tema01/img31.png)
Como vemos en la captura, podemos comprobar la asignación de la IP, utilizando el comando:

```
ip/address/print
```
![img02](tema01/img32.png)


## Asignación manual de ruta por defecto

Si deseamos poder navegar por internet, deberemos indicar cuál es la puerta de enlace de la red, añadiendo una ruta por defecto:

```
ip/route/add dst-address=0.0.0.0/0 gateway=192.168.122.1 comment="Salida a NAT"
```

![img02](tema01/img33.png)

## Asignación manual de servidores DNS

Y si se requiere traducción de nombres de dominio, deberemos configurar las IP de los servidores DNS:

```
ip/dns/set servers=8.8.8.8,8.8.4.4 allow-remote-requests=yes
```
![img02](tema01/img34.png)
Con esta configuración, el router tendría acceso a internet. Podemos comprobarlo accediendo a una URL conocida, con el siguiente comando:

```
ping google.com
```
![img02](tema01/img35.png)
También podemos acceder al router utilizando un navegador web de la máquina anfitrión, a través de la IP configurada:

![img02](tema01/img36.png)

## Eliminación manual de la configuración aplicada

Para eliminar la configuración aplicada, realizamos las siguientes operaciones:

- Eliminamos las direcciones de los servidores DNS

```
ip/dns/set servers=""
```
![img02](tema01/img37.png)
- Visualizamos las rutas existentes, y eliminamos la ruta por defecto creada

```
ip/route/print
ip/route/remove <indice>
```
![img02](tema01/img38.png)
- Eliminamos la IP de la interfaz física ether1

```
ip/address/print
ip/address/remove <indice>
```
![img02](tema01/img39.png)


Tras eliminar los elementos configurados, volvemos a tener un router en blanco, preparado para la próxima práctica.

#Preguntas

### 1. ¿Qué es una IP estática?
<details>
<summary>Ver respuesta</summary>

Es una dirección IP configurada manualmente en un dispositivo y que no cambia automáticamente.

</details>

### 2. ¿En qué casos se utiliza IP estática?
<details>
<summary>Ver respuesta</summary>

En servidores, routers, gateways o entornos donde se requiere una configuración fija y controlada.

</details>

### 3. ¿Qué diferencia hay entre IP estática y dinámica?
<details>
<summary>Ver respuesta</summary>

La IP estática se configura manualmente, mientras que la dinámica se obtiene automáticamente mediante DHCP.

</details>

### 4. ¿Por qué un router puede necesitar IP estática?
<details>
<summary>Ver respuesta</summary>

Para actuar como gateway estable dentro de una red y garantizar conectividad constante.

</details>

### 5. ¿Qué comando muestra las interfaces del router?
<details>
<summary>Ver respuesta</summary>

interface/print

</details>

### 6. ¿Cómo se asigna una IP estática a una interfaz?
<details>
<summary>Ver respuesta</summary>

ip/address/add address=192.168.122.61/24 interface=ether1

</details>

### 7. ¿Cómo se comprueba la IP configurada?
<details>
<summary>Ver respuesta</summary>

ip/address/print

</details>

### 8. ¿Qué comando añade una ruta por defecto?
<details>
<summary>Ver respuesta</summary>

ip/route/add dst-address=0.0.0.0/0 gateway=192.168.122.1

</details>

### 9. ¿Para qué sirve la ruta por defecto?
<details>
<summary>Ver respuesta</summary>

Permite que el router envíe tráfico hacia redes externas (Internet).

</details>
### 10. ¿Cómo se configuran los servidores DNS?
<details>
<summary>Ver respuesta</summary>

ip/dns/set servers=8.8.8.8,8.8.4.4 allow-remote-requests=yes

</details>

### 11. ¿Para qué sirven los DNS?
<details>
<summary>Ver respuesta</summary>

Para traducir nombres de dominio en direcciones IP.

</details>

### 12. ¿Cómo se comprueba la conectividad a Internet?
<details>
<summary>Ver respuesta</summary>

Haciendo un ping, por ejemplo:

ping google.com

</details>

### 13. ¿Qué significa que el ping funcione correctamente?
<details>
<summary>Ver respuesta</summary>

Que el router tiene conectividad a Internet y resolución DNS correcta.

</details>
### 14. ¿Qué pasaría si configuras la IP pero no la ruta por defecto?
<details>
<summary>Ver respuesta</summary>

El router solo podrá comunicarse dentro de su red local, pero no tendrá acceso a Internet.

</details>

### 15. ¿Qué ocurre si no configuras los DNS?
<details>
<summary>Ver respuesta</summary>

Habrá conectividad IP, pero no se podrán resolver nombres de dominio (no funcionarán URLs).

</details>

### 16. ¿Por qué es importante configurar todos los elementos (IP, ruta, DNS)?
<details>
<summary>Ver respuesta</summary>

Porque cada uno cumple una función: la IP identifica el equipo, la ruta permite salir de la red y el DNS permite resolver nombres.

</details>
### 17. ¿Cómo se eliminan los servidores DNS?
<details>
<summary>Ver respuesta</summary>

ip/dns/set servers=""

</details>

### 18. ¿Cómo se elimina una ruta?
<details>
<summary>Ver respuesta</summary>

ip/route/print  
ip/route/remove <indice>

</details>

### 19. ¿Cómo se elimina una IP de una interfaz?
<details>
<summary>Ver respuesta</summary>

ip/address/print  
ip/address/remove <indice>

</details>

### 20. ¿Qué ocurre tras eliminar toda la configuración?
<details>
<summary>Ver respuesta</summary>

El router queda sin configuración de red, listo para una nueva práctica.

</details>
!!! warning
    No olvides configurar la ruta por defecto o no tendrás acceso a Internet.

!!! danger
    La IP, la ruta y el DNS deben estar correctamente configurados para que todo funcione.

!!! tip
    Usa siempre ping para verificar que la configuración es correcta.