# Virtualización de RouterOS con GNS3.
Jorge López Fernández  
2025/12/26  

---

# Índice

Introducción  
Configuración de una plantilla reutilizable de RouterOS-7.20.6 en GNS3  
Descarga de la imagen ISO de RouterOS  
Creación de la plantilla RouterOS-7.20.6  
Creación de nuestro primer proyecto.  
Configuración del escenario  
Arranque del dispositivo  
Conexión a la consola.  
Instalación de RouterOS.  
Modificación de la plantilla base.  

---

# Introducción

En el documento anterior hemos trabajado la instalación de RouterOS utilizando VirtualBox. Trabajar con VirtualBox es una aproximación muy adecuada para los primeros contactos con MikroTik en un entorno sencillo y controlado.

Sin embargo, a medida que avanzamos en el aprendizaje, aparece una nueva necesidad: construir escenarios de red más complejos, donde intervengan varios routers, switches y equipos finales, y donde la topología tenga un peso tan importante como la propia configuración de los dispositivos. 

Es en este punto donde entrará en juego GNS3, entorno de simulación y emulación de redes pensado específicamente para el diseño de laboratorios. Su enfoque permitirá visualizar de forma clara cómo se interconectan los dispositivos, facilitando una comprensión mucho más profunda de conceptos como el enrutamiento, la segmentación de redes o el flujo del tráfico.

Conviene aclarar, antes de continuar, el alcance de este bloque. En este apartado nos centraremos exclusivamente en los pasos necesarios para desplegar RouterOS de MikroTik dentro de GNS3, asumiendo que el entorno base de GNS3 ya está instalado y operativo. 

No se profundizará en la instalación ni en la configuración general de GNS3, ya que esos contenidos se abordan de forma específica en otro curso dedicado íntegramente a esta herramienta. De este modo, el foco se mantiene en MikroTik y en su integración dentro del laboratorio de red, evitando duplicar contenidos y permitiendo avanzar de forma más ágil y coherente.

---

# Configuración de una plantilla reutilizable de RouterOS-7.20.6 en GNS3 

## Descarga de la imagen ISO de RouterOS

Vamos a volver a utilizar la imagen ISO de RouterOS, en lugar de la versión CHR (Cloud Hosted Router) en formato RAW disk, para generar una plantilla limpia de la que partir, tras la instalación.

La descarga debe realizarse siempre desde la web oficial de MikroTik, por dos motivos fundamentales que conviene remarcar en un contexto educativo:

• Garantizamos que el software es auténtico y actualizado.  
• Evitamos problemas de seguridad derivados de repositorios no oficiales.  

Para ello, accedemos a la URL https://mikrotik.com/download, seleccionamos la arquitectura X86, y descargamos la última versión de la imagen ISO correspondiente (durante la redacción de este manual, descargamos la versión 7.20.6).

![Descarga RouterOS](img/tema2/01_descarga.png)

---

## Creación de la plantilla RouterOS-7.20.6

En este punto vamos a dar un paso especialmente importante desde el punto de vista práctico y organizativo: la creación de una plantilla reutilizable de RouterOS. Trabajar con plantillas nos permite ahorrar tiempo, evitar configuraciones repetitivas y mantener coherencia entre los distintos laboratorios que se vayan creando.

Al abrir GNS3 por primera vez, nos solicitará crear un nuevo proyecto. Pulsaremos el botón Cancelar.

![Cancelar proyecto](img/tema2/02_cancelar.png)

Accede a la configuración de GNS, a través de la opción Preferences del menú Edit. 

![Preferences](img/tema2/03_preferences.png)

En el menú lateral, selecciona la opción QEMU VMs dentro de QEMU, y pulsa la opción New, para crear una nueva máquina virtual.

![QEMU VMs](img/tema2/04_qemu.png)

Se abrirá un asistente, para generar la nueva plantilla. En el primer paso debemos introducir el nombre de la plantilla. Para identificar la versión del router, utilizaremos en el nombre la versión de RouterOS utilizada (RouterOS-7.20.6), y pulsaremos el botón Siguiente.

![Nombre plantilla](img/tema2/05_nombre.png)

En la siguiente ventana del asistente, dejaremos los parámetros por defecto, y pulsaremos el botón siguiente.

![Parametros](img/tema2/06_parametros.png)

En la siguiente ventana del asistente, seleccionaremos como consola vnc, y pulsaremos el botón siguiente.

![VNC](img/tema2/07_vnc.png)

En la siguiente ventana del asistente, marcamos la opción New Image.

![New Image](img/tema2/08_new_image.png)

Pulsamos el botón Create, y en el asistente, dejamos los parámetros por defecto.

![Create](img/tema2/09_create.png)

Pulsamos el botón siguiente, y volvemos a dejar los parámetros por defecto.

![Parametros disco](img/tema2/10_disco.png)

Indicamos el nombre que queremos dar al disco de nuestro router plantilla, y pulsamos el botón finaliza. 

![Nombre disco](img/tema2/11_nombre_disco.png)

De vuelta a la ventana principal del asistente, pulsamos de nuevo el botón finalizar.

---

Dado que hemos creado una plantilla con un disco en blanco, necesitaremos inicializar el disco. Para ello, debemos configurar la plantilla con la imagen ISO descargada, para proceder a la instalación. En primer lugar, entramos en la edición de la plantilla, pulsando el botón Edit. 

![Editar plantilla](img/tema2/12_edit.png)

Marcamos como opción principal de arranque la unidad de CD.

![Boot CD](img/tema2/13_boot_cd.png)

También podemos etiquetar la plantilla como un router, y asignarle el icono correspondiente.

![Icono](img/tema2/14_icono.png)

Seleccionamos la ISO como unidad de CD.

![ISO](img/tema2/15_iso.png)

Configuramos la plantilla para que utilice 4 interfaces de red.

![Interfaces](img/tema2/16_interfaces.png)

Y desmarcamos la opción “Use as a linked base VM”, para que los cambios aplicados durante la primera ejecución (la instalación) se persistan en el disco de la plantilla original.

![Linked VM](img/tema2/17_linked.png)

Pulsamos el botón Aceptar y tendremos lista nuestra plantilla para reutilizar en todos los escenarios que configuremos.

![Plantilla lista](img/tema2/18_lista.png)

---

** Asegúrate de tener un cliente VNC instalado en el sistema, para poder acceder a la consola del Router, durante la ejecución de los proyectos. Para instalar un cliente VNC desde consola, en un sistema basado en Ubuntu, puedes ejecutar: 

sudo apt update;  
sudo apt install tigervnc-viewer;  

---

# Creación de nuestro primer proyecto.

Para crear nuestro primer proyecto, accedemos al menú File, de GNS3, y pulsamos la opción New blank Project.

![Nuevo proyecto](img/tema2/19_new_project.png)

A continuación, rellenamos el nombre del proyecto. Para el ejemplo dado, utilizaremos el nombre “prueba-mikrotik-001”, y pulsamos el botón “Aceptar”.

![Nombre proyecto](img/tema2/20_nombre.png)

GNS3 nos mostrará un escenario vacío, al que podremos ir añadiendo nuestros routers y equipos, para simular un entorno de red. 

---

# Configuración del escenario.

Vamos a arrastrar nuestro Router al escenario, accediendo al menú Routers, y arrastrándolo nuestro RouterOS al escenario.

![Router](img/tema2/21_router.png)

A continuación, vamos a arrastrar un elemento NAT al escenario, para conectar el router a la red física. 

![NAT](img/tema2/22_nat.png)

Por último, conectaremos el puerto 0 del router al puerto 0 del elemento Cloud, utilizando un cable.

![Conexion](img/tema2/23_conexion.png)

Nuestro escenario está listo para probar.

---

# Arranque del dispositivo.

Para arrancar el escenario, pulsamos el botón PLAY de GNS3. La aplicación nos preguntará si queremos arrancar todos los dispositivos. Le indicamos que sí, y esperamos a que finalice el arranque.

![Play](img/tema2/24_play.png)

En el caso de que no queramos encender todos los dispositivos, cuando tenemos un escenario más complejo, podemos pulsar el botón derecho del ratón sobre el elemento que queremos iniciar/parar/pausar, y seleccionar la opción correspondiente.

![Opciones](img/tema2/25_opciones.png)

---

# Conexión a la consola.

Para acceder a la consola de un dispositivo, debemos seleccionar el mismo, y pulsar el botón Console, de la barra de herramientas de GNS3.

También podemos pulsar el botón derecho del ratón en el elemento al que queremos acceder, y seleccionar la opción Console, del menú desplegable.

![Console](img/tema2/26_console.png)

En ambos casos, se nos mostrará un terminal o escritorio remoto, conectado y listo para interactuar con el elemento.

---

# Instalación de RouterOS.

Como vemos, durante el primer arranque, nos solicitará instalar el sistema operativo.

![Instalacion](img/tema2/27_instalacion.png)

Para ello, podemos pulsar la siguiente secuencia de teclas:

- a (instalar todo)  
- Flecha abajo (marca todos los elementos)  
- i (inicia el proceso de instalación)  
- y (confirmamos la acción, para que inicie)  

Se iniciará la instalación del sistema operativo RouterOS en la plantilla. Esperaremos hasta que nos indique que ha finalizado, y nos pida permiso para reiniciar.

![Proceso](img/tema2/28_proceso.png)

Pulsamos “ENTER” para reiniciar el router. Dado que la plantilla está configurada para arrancar desde el CD, volverá a iniciarse la instalación. Detenemos la ejecución, pulsando sobre el botón STOP.

---

# Modificación de la plantilla base.

Seleccionamos el router, en el escenario.

![Seleccion](img/tema2/29_seleccion.png)

Y pulsamos la tecla Suprimir, para eliminarlo del proyecto. 

![Eliminar](img/tema2/30_eliminar.png)

Tras eliminarlo, vamos a volver a modificar la plantilla, entrando en el menú Edit → Preferences.

![Preferences](img/tema2/31_preferences.png)

Seleccionamos la plantilla, y pulsamos el botón editar.

![Editar](img/tema2/32_editar.png)

Modificamos la prioridad de arranque, para indicar que arranque desde el disco duro.

![Boot disco](img/tema2/33_boot.png)

Eliminamos la imagen ISO del CD.

![Eliminar ISO](img/tema2/34_iso.png)

Y volvemos a marcar la opción “Use as a linked base VM”, para que se cree una copia del disco para cada instancia de la plantilla.

![Linked final](img/tema2/35_final.png)

---

Nuestra plantilla está lista para ser utilizada en todos los proyectos, con una instalación limpia.

Podemos realizar la prueba, arrastrándola de nuevo al proyecto e iniciando el router, que nos solicitará las credenciales de login.

Entramos con usuario admin, sin contraseña (pulsando intro cuando nos solicite la contraseña).

El router nos solicitará si deseamos leer la licencia de software. Podemos leerla, o indicarle que no, pulsando la tecla ‘n’.

Tras este paso, nos solicitará la nueva contraseña de configuración (que deberemos recordar, para poder administrar el router).

Ya disponemos de un router limpio sobre el que trabajar en los próximos tutoriales.