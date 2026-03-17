# Virtualización de RouterOS con GNS3  
**Jorge López Fernández**  
2025/12/26  

---

# Índice

- Introducción  
- Configuración de una plantilla reutilizable de RouterOS en GNS3  
  - Descarga de la imagen ISO  
  - Creación de la plantilla  
- Creación de un proyecto  
- Configuración del escenario  
- Arranque del dispositivo  
- Conexión a la consola  
- Instalación de RouterOS  
- Modificación de la plantilla  

---

# Introducción

En un primer momento, RouterOS puede trabajarse con VirtualBox en un entorno sencillo.

Sin embargo, cuando se necesitan escenarios más complejos con varios dispositivos, aparece la necesidad de utilizar herramientas específicas de simulación de redes.

GNS3 permite:

- Diseñar topologías de red  
- Conectar múltiples dispositivos  
- Visualizar la estructura de la red  
- Comprender el flujo del tráfico  

En este documento se explica cómo desplegar RouterOS en GNS3, asumiendo que GNS3 ya está instalado.

---

# Configuración de una plantilla reutilizable de RouterOS en GNS3

## Descarga de la imagen ISO

Se utilizará la imagen ISO de RouterOS en arquitectura x86.

Descarga desde la web oficial:

https://mikrotik.com/download

Motivos:

- Software actualizado  
- Seguridad  
- Evitar fuentes no oficiales  

---

## Creación de la plantilla

1. Abrir GNS3  
2. Cancelar la creación inicial de proyecto  
3. Ir a:

   Edit → Preferences → QEMU → QEMU VMs  

4. Pulsar **New**

### Configuración inicial

- Nombre: `RouterOS-7.20.6`  
- Mantener valores por defecto  
- Consola: VNC  
- Seleccionar **New Image**  

### Creación del disco

- Crear disco (qcow2)  
- Mantener parámetros por defecto  
- Asignar nombre al disco  
- Finalizar  

---

## Configuración de la plantilla

Editar la plantilla:

- Arranque desde CD  
- Asignar categoría: Router  
- Seleccionar la ISO  
- Configurar 4 interfaces de red  
- Desmarcar: `Use as a linked base VM`  

Guardar configuración.

---

# Creación de un proyecto

1. File → New blank project  
2. Nombre: `prueba-mikrotik-001`  
3. Crear escenario vacío  

---

# Configuración del escenario

1. Arrastrar router RouterOS al escenario  
2. Añadir elemento NAT  
3. Conectar:

   - Puerto 0 del router  
   - Puerto 0 del NAT  

El escenario queda preparado.

---

# Arranque del dispositivo

- Pulsar botón **Play**  
- Confirmar arranque de dispositivos  

Opcional:

- Arranque individual con clic derecho  
- Opciones: Start / Stop / Suspend  

---

# Conexión a la consola

Opciones:

- Botón Console  
- Clic derecho → Console  

Se abrirá una terminal conectada al dispositivo.

---

# Instalación de RouterOS

En el primer arranque, se inicia la instalación.

Secuencia de teclas:

- a → instalar todo  
- Flecha abajo → seleccionar paquetes  
- i → iniciar instalación  
- y → confirmar  

Esperar a finalizar.

Pulsar ENTER para reiniciar.

Detener el sistema (STOP) para evitar reinstalación.

---

# Modificación de la plantilla

1. Eliminar el router del proyecto  
2. Editar plantilla:

   Edit → Preferences  

3. Cambios:

- Arranque desde disco duro  
- Eliminar ISO  
- Activar `Use as a linked base VM`  

Guardar.

---

# Uso de la plantilla

Al reutilizar la plantilla:

- Router ya instalado  
- Acceso inicial:

  - Usuario: admin  
  - Sin contraseña  

Primer inicio:

- Opción licencia: n  
- Definir nueva contraseña  

---

Plantilla lista para utilizar en nuevos escenarios.
