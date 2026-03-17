# Conectando a RouterOS para su administración  
---

# Introducción

Una vez creada y puesta en marcha la instancia de RouterOS, ya sea en VirtualBox o en GNS3, el siguiente paso es acceder al router para empezar a configurarlo.

RouterOS permite varias formas de administración:

- Consola  
- WinBox  
- WebFig  
- SSH  

---

# Estado inicial del router

En el primer acceso, RouterOS se presenta sin configuraciones previas:

- No hay direcciones IP  
- No hay reglas  
- No hay servicios configurados  

El sistema está listo para comenzar desde cero.

---

# Usuario y contraseña por defecto

Credenciales iniciales:

- Usuario: `admin`  
- Contraseña: (vacía)  

---

# Acceso por consola

Es el método más básico y fiable.

- No requiere red  
- Acceso directo al sistema  
- Usado en instalación y recuperación  

En entornos virtualizados:

- Consola de VirtualBox  
- Consola de GNS3  

![Consola RouterOS](img/tema3/consola_routeros.png)

---

# Primer acceso

1. Usuario: `admin`  
2. Contraseña: (vacía)  

![Login RouterOS](img/tema3/login_routeros.png)

Durante el primer acceso:

- `y` → ver licencia  
- `n` → continuar  

![Licencia RouterOS](img/tema3/licencia_routeros.png)

---

# Cambio de contraseña

RouterOS obliga a definir una nueva contraseña.

![Cambio contraseña](img/tema3/cambio_password.png)

---

# Comprobación de IP

Comando: 
/ip/address/print


Muestra las direcciones IP del router.

![Comando IP](img/tema3/comando_ip.png)

---

## Acceso con WinBox

WinBox es la herramienta gráfica oficial de MikroTik.

Descarga:
https://mikrotik.com/download/winbox

![Descarga WinBox](img/tema3/winbox_descarga.png)

---

### Escaneo de red

WinBox detecta dispositivos automáticamente:

- Por IP  
- Por MAC  

![WinBox scan](img/tema3/winbox_scan.png)

---

### Conexión

1. Seleccionar dispositivo  
2. Introducir usuario y contraseña  
3. Conectar  

![WinBox login](img/tema3/winbox_login.png)

---

## Acceso con WebFig

Interfaz web de RouterOS.

Requisitos:

- IP configurada  
- Navegador  

![WebFig interfaz](img/tema3/webfig_interfaz.png)

---

### Login WebFig

![WebFig login](img/tema3/webfig_login.png)

---

### Panel WebFig

![WebFig panel](img/tema3/webfig_panel.png)

---

## Acceso mediante SSH

Permite administración remota en modo texto.

![SSH conexión](img/tema3/ssh_conexion.png)

---

### Sesión SSH

![SSH terminal](img/tema3/ssh_terminal.png)

---

RouterOS puede administrarse mediante distintos métodos, cada uno adecuado a una situación concreta.