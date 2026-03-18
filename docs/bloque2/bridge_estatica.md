# Configuración de IP estática sobre bridge.

## Índice

- Introducción
- Ejemplo 4 — Configuración de IP estática sobre un bridge
  - Creación del bridge Gateway
  - Asignación de la interfaz física al bridge
  - Asignación manual de IP al bridge
  - Asignación manual de ruta por defecto
  - Asignación manual de servidores DNS
- Eliminando la configuración aplicada

---

## Introducción

A lo largo de esta práctica volveremos a trabajar sobre la configuración de la interfaz que actuará como gateway, aplicando una configuración de red estática sobre un bridge.

Este cuarto escenario representa la evolución del primer escenario, aportando resiliencia en caso de fallo del puerto físico.

---

## Ejemplo 4 — Configuración de IP estática sobre un bridge

Durante este ejemplo, se creará un bridge, al que añadiremos la primera interfaz del router. Posteriormente, aplicaremos una configuración de red estática sobre el bridge, y se verificará que el router puede comunicarse correctamente a través del bridge, utilizando la configuración de red aplicada sobre el mismo.

---

## Creación del bridge Gateway

En primer lugar, vamos a listar los bridges configurados en nuestro router, ejecutando el siguiente comando:

```sh
interface/bridge/print
```

Podemos observar cómo no hay ningún bridge configurado por defecto.
![img02](tema01/img60.png)

Vamos a crear el bridge que actuará como Gateway de la red, ejecutando el siguiente comando:

```sh
interface/bridge/add name=bridge-gateway comment="Bridge Gateway"
```
![img02](tema01/img61.png)
Verificamos que todo ha ido bien:

```sh
interface/bridge/print
```
![img02](tema01/img62.png)
---

## Asignación de la interfaz física al bridge

Vamos a listar los bridges configurados en nuestro router, ejecutando el siguiente comando:

```sh
interface/bridge/print
```
![img02](tema01/img63.png)
Añadimos la interfaz ether1 al bridge “bridge-gateway”:

```sh
interface/bridge/port/add bridge=bridge-gateway interface=ether1
```
![img02](tema01/img64.png)
Una vez añadida al bridge, **ether1 deja de ser el punto lógico de red**. El punto lógico pasa a ser el bridge.

Verificamos que todo ha ido bien:

```sh
interface/bridge/port/print
```
![img02](tema01/img65.png)
---

## Asignación manual de IP al bridge

Para finalizar el ejemplo, vamos a asignar una IP al bridge, de manera manual.

Primero listamos los bridges configurados en nuestro router, ejecutando el siguiente comando:

```sh
interface/bridge/print
```
![img02](tema01/img66.png)
En mi caso, configuraré el bridge “bridge-gateway”, asignándole la IP 192.168.122.61/24, ejecutando:

```sh
ip/address/add address=192.168.122.61/24 interface=bridge-gateway comment="IP Bridge gateway"
```
![img02](tema01/img67.png)
Podemos comprobar la asignación de la IP, utilizando el comando:

```sh
ip/address/print
```
![img02](tema01/img68.png)
---

## Asignación manual de ruta por defecto

Si deseamos poder navegar por internet, deberemos indicar cuál es la puerta de enlace de la red, añadiendo una ruta por defecto:

```sh
ip/route/add dst-address=0.0.0.0/0 gateway=192.168.122.1 comment="Salida a NAT"
```
![img02](tema01/img69.png)
---

## Asignación manual de servidores DNS

Si se requiere traducción de nombres de dominio, deberemos configurar las IP de los servidores DNS:

```sh
ip/dns/set servers=8.8.8.8,8.8.4.4 allow-remote-requests=yes
```
![img02](tema01/img70.png)

Con esta configuración, el router tendría acceso a internet. Podemos comprobarlo ejecutando un ping sobre un dominio conocido:

```sh
ping google.com
```
![img02](tema01/img71.png)
También podemos acceder al router utilizando un navegador web de la máquina anfitrión, a través de la IP configurada.
![img02](tema01/img73.png)
---

## Eliminando la configuración aplicada

Para eliminar la configuración aplicada, realizamos las siguientes operaciones:

### Eliminamos las direcciones de los servidores DNS

```sh
ip/dns/set servers=""
```
![img02](tema01/img74.png)
---

### Visualizamos las rutas existentes y eliminamos la ruta por defecto creada

```sh
ip/route/print
ip/route/remove <indice>
```
![img02](tema01/img74.png)
---

### Eliminamos la interfaz del bridge

```sh
interface/bridge/port/print
interface/bridge/port/remove <indice>
```
![img02](tema01/img75.png)
---

### Eliminamos la IP del bridge

```sh
ip/address/print
ip/address/remove <indice>
```
![img02](tema01/img76.png)

---

### Por último, eliminaremos el bridge

```sh
interface/bridge/print
interface/bridge/remove <indice>
```
![img02](tema01/img78.png)
---

Tras eliminar los elementos configurados, volvemos a tener un router en blanco, preparado para la próxima práctica.
