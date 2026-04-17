# Preguntas y soluciones — Firewall en MikroTik (RouterOS)

Repaso completo basado en el tema de configuración de reglas de firewall en MikroTik, organizado por bloques de dificultad creciente.

---

## Bloque 1 — Conceptos básicos

**1. ¿Cuáles son los tres elementos básicos que definen una regla de firewall en MikroTik?**

??? success "Ver solución"
    Los tres elementos son:

    - **Chain (cadena):** indica en qué punto del recorrido del tráfico se aplica la regla (`input`, `forward` u `output`). Es obligatorio.
    - **Action (acción):** define qué hace el firewall si el tráfico coincide (`accept`, `drop`, `reject`, `log`). Es obligatorio.
    - **Condiciones (filtros):** especifican qué tráfico debe coincidir (interfaz, IP, protocolo, puerto, estado de conexión…). Son opcionales, pero si no se define ninguna, la regla se aplica a **todo** el tráfico de esa cadena.

---

**2. ¿Qué cadena usarías para filtrar el acceso SSH al propio router? ¿Y para filtrar tráfico entre la LAN e Internet?**

??? success "Ver solución"
    - Acceso SSH **al propio router** → cadena `input` (el destino es el dispositivo MikroTik).
    - Tráfico **que atraviesa** el router (LAN → Internet) → cadena `forward`.

    > La cadena `output` se usaría para controlar el tráfico generado por el propio router, algo menos habitual.

---

**3. ¿Qué diferencia hay entre las acciones `drop` y `reject`?**

??? success "Ver solución"
    | Acción | Comportamiento |
    |--------|----------------|
    | `drop` | Bloquea el tráfico **silenciosamente**, sin notificar al origen. |
    | `reject` | Bloquea el tráfico **y envía un mensaje de rechazo** al origen. |

    `drop` es la opción recomendada en entornos con Internet: no revela información sobre la existencia del firewall ni sobre el estado del puerto.

---

**4. ¿Qué ocurre en MikroTik si un paquete no coincide con ninguna regla de la cadena?**

??? success "Ver solución"
    El tráfico **se permite**. MikroTik **no tiene política por defecto implícita** de bloqueo, a diferencia de firewalls como UFW. Por eso, una configuración segura debe finalizar siempre con una regla explícita de bloqueo general (`action=drop` sin condiciones).

---

## Bloque 2 — Procesamiento y orden de reglas

**5. Un router tiene estas reglas en `forward` en este orden: (1) drop src=192.168.1.0/24 y (2) accept src=192.168.1.10. ¿Qué ocurre con los paquetes de 192.168.1.10?**

??? success "Ver solución"
    Los paquetes de `192.168.1.10` serán **bloqueados** por la regla 1, ya que esa IP pertenece a la red `192.168.1.0/24`. El procesamiento es secuencial: en cuanto una regla coincide, se ejecuta su acción y se detiene la evaluación. La regla 2 nunca llega a evaluarse.

    !!! warning "Conclusión"
        El orden de las reglas es crítico. Las reglas más específicas deben colocarse siempre **antes** que las más generales.

---

**6. ¿Por qué es buena práctica colocar la regla `connection-state=established,related action=accept` al principio de la cadena?**

??? success "Ver solución"
    Por dos motivos:

    1. **Funcional:** garantiza que las respuestas a conexiones ya iniciadas (por ejemplo, respuestas HTTP desde Internet hacia la LAN) sean aceptadas sin necesidad de reglas adicionales.
    2. **Rendimiento:** al estar al principio, la mayoría del tráfico legítimo (que suele ser `established`) se procesa con la primera regla, sin evaluar el resto, reduciendo la carga de CPU del router.

---

**7. ¿Qué significa que un paquete tenga `connection-state=invalid`?**

??? success "Ver solución"
    Significa que RouterOS **no puede asociar ese paquete a ninguna conexión válida** en su tabla de seguimiento de conexiones. Puede deberse a:

    - Paquetes llegados fuera de orden.
    - Errores de protocolo o cabeceras inconsistentes.
    - Paquetes sin relación con ninguna sesión existente.

    Estos paquetes deben descartarse con `action=drop` porque pueden indicar un ataque o un fallo de red.

---

**8. ¿Qué diferencia hay entre los estados `established` y `related`?**

??? success "Ver solución"
    | Estado | Descripción |
    |--------|-------------|
    | `established` | Paquetes que pertenecen a una conexión ya iniciada (por ejemplo, respuestas HTTP a una petición de la LAN). |
    | `related` | Paquetes asociados a una conexión existente pero que forman parte de un flujo secundario (por ejemplo, transferencias de datos FTP o respuestas ICMP relacionadas con una conexión TCP). |

    Ambos se suelen permitir juntos con la misma regla al inicio de la cadena.

---

## Bloque 3 — Comandos y configuración

**9. Escribe el comando para añadir una regla que permita todo el tráfico desde la interfaz `bridge-lan` hacia `ether1` en la cadena `forward`.**

??? success "Ver solución"
    ```
    ip/firewall/filter/add chain=forward \
      in-interface=bridge-lan \
      out-interface=ether1 \
      action=accept \
      comment="Permitir tráfico LAN a Internet"
    ```

    - `in-interface`: filtra por la interfaz de **entrada** del paquete.
    - `out-interface`: filtra por la interfaz de **salida** del paquete.

---

**10. ¿Cómo listarías las reglas actuales del firewall y cómo eliminarías la regla con índice 2?**

??? success "Ver solución"
    Listar todas las reglas:

    ```
    ip/firewall/filter/print
    ```

    Eliminar la regla en el índice 2:

    ```
    ip/firewall/filter/remove 2
    ```

    !!! warning "Precaución"
        Siempre lista las reglas antes de eliminar. Los índices son dinámicos y pueden cambiar al añadir o quitar reglas.

---

**11. ¿Para qué sirve el parámetro `place-before` al añadir una regla? ¿Cuándo es imprescindible usarlo?**

??? success "Ver solución"
    `place-before=N` inserta la nueva regla en la posición N, desplazando las siguientes hacia abajo.

    Es imprescindible cuando ya existe una regla de **bloqueo general** al final de la cadena: si añades una regla de permiso sin especificar posición, se añade al final (después del bloqueo) y nunca llega a evaluarse.

    Ejemplo: insertar una regla de permiso antes del bloqueo general que ocupa la posición 2:

    ```
    ip/firewall/filter/add chain=forward \
      in-interface=bridge-lan \
      out-interface=ether1 \
      action=accept \
      place-before=2 \
      comment="Permitir tráfico LAN a Internet"
    ```

---

**12. ¿Cuál es la diferencia entre filtrar el tráfico por `src-address=10.10.0.0/24` frente a `in-interface=bridge-lan`?**

??? success "Ver solución"
    - `src-address`: filtra por la **dirección IP de origen** del paquete. Es más preciso cuando quieres controlar una subred concreta, independientemente de la interfaz.
    - `in-interface`: filtra por la **interfaz física o lógica** por la que entra el paquete. Es más flexible cuando el rango de IPs puede cambiar pero la topología de red es estable.

    En redes bien diseñadas ambos criterios suelen coincidir, pero `in-interface` es más robusto frente a cambios de direccionamiento.

---

## Bloque 4 — Casos prácticos

**13. Un administrador configura estas reglas en `forward` en este orden: (1) accept established/related, (2) drop invalid, (3) accept src=10.10.0.0/24 out-interface=ether1, (4) drop sin condiciones. ¿Qué ocurrirá con una respuesta HTTP que llega desde Internet hacia un cliente de la LAN?**

??? success "Ver solución"
    La respuesta HTTP pertenece a una conexión ya establecida (fue el cliente quien la inició). Por tanto, coincidirá con la **regla 1** (`established/related → accept`) y será aceptada. El procesamiento se detiene ahí; las reglas 2, 3 y 4 no se evalúan para ese paquete.

    Este es precisamente el beneficio de colocar la regla `established/related` al inicio.

---

**14. ¿Qué problema tendría esta configuración? (1) drop sin condiciones, (2) accept established/related, (3) accept src=10.10.0.0/24**

??? success "Ver solución"
    La regla 1 bloquea **absolutamente todo el tráfico** antes de que pueda evaluarse ninguna otra regla. Las reglas 2 y 3 nunca se alcanzan. El resultado: el firewall deniega toda comunicación, incluida la legítima.

    !!! danger "Regla de oro"
        La regla de bloqueo general (`drop` sin condiciones) debe ir siempre al **final** de la cadena, nunca al principio.

---

**15. Tienes la configuración de firewall en funcionamiento y un administrador ejecuta `ip/firewall/filter/remove 0` por error. ¿Qué consecuencias puede tener?**

??? success "Ver solución"
    La regla 0 era `accept connection-state=established,related`. Al eliminarla:

    - Las **respuestas a conexiones ya iniciadas** dejan de ser aceptadas automáticamente.
    - Navegación web, sesiones SSH, consultas DNS y cualquier tráfico de respuesta empezarán a fallar.
    - Los paquetes de respuesta llegarán a la regla de bloqueo general y serán descartados.

    La solución es volver a añadir la regla cuanto antes y colocarla en la posición 0 con `place-before=0`.

---

## Resumen — Política de firewall recomendada para `forward`

```
# 1. Permitir conexiones ya establecidas o relacionadas
ip/firewall/filter/add chain=forward \
  connection-state=established,related \
  action=accept \
  comment="Permitir conexiones establecidas y relacionadas"

# 2. Bloquear tráfico inválido
ip/firewall/filter/add chain=forward \
  connection-state=invalid \
  action=drop \
  comment="Bloquear tráfico inválido"

# 3. Permitir explícitamente el tráfico legítimo
ip/firewall/filter/add chain=forward \
  in-interface=bridge-lan \
  out-interface=ether1 \
  action=accept \
  comment="Permitir LAN a Internet"

# 4. Bloquear el resto — SIEMPRE al final
ip/firewall/filter/add chain=forward \
  action=drop \
  comment="Bloquear todo el resto del tráfico"
```

| Posición | Regla | Motivo |
|----------|-------|--------|
| 0 | `established/related → accept` | Rendimiento y funcionalidad |
| 1 | `invalid → drop` | Seguridad |
| 2–N | Reglas específicas de permiso | Política de red |
| Último | `drop` sin condiciones | Cierre explícito de la política |
