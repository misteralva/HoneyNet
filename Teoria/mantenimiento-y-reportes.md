# Mantenimiento y reportes

## En simple

### Por qué no basta con instalar

Desplegar un sistema de seguridad es solo el principio. Un honeypot sin mantenimiento acaba mal: se queda desactualizado, se llena el disco de logs, deja de enviar datos sin que nadie se entere, o se compromete y se convierte en un riesgo para el cliente.

> **Analogía:** instalar un honeypot es como comprar un coche; mantenerlo es llevarlo a revisión, cambiar el aceite y mirar los pilotos. Y el informe es la ficha de revisión que se entrega al propietario: qué se ha hecho, cómo está y qué conviene vigilar.

El servicio completo se apoya en tres ideas:

- **Mantener:** que todo siga funcionando, seguro y actualizado.
- **Vigilar:** enterarse cuando algo falla o se comporta de forma rara.
- **Reportar:** explicar al cliente, con datos, qué ha pasado.

### Para quién es un informe

Un buen informe tiene dos tipos de lectores:

- **Directivo o responsable del cliente:** quiere un resumen claro, sin jerga, con lo importante y qué decisiones tomar.
- **Técnico del cliente:** quiere detalle (IPs, técnicas, evidencias) para actuar.

Por eso el informe suele empezar con un **resumen ejecutivo** y dejar el detalle técnico para después.

---

## Conceptos clave

| Término | Significado |
|---|---|
| **Mantenimiento preventivo** | Tareas periódicas para evitar fallos (actualizar, comprobar copias...). |
| **Mantenimiento correctivo** | Reparar algo que ya ha fallado. |
| **Monitorización** | Comprobar de forma continua que los servicios funcionan. |
| **Retención** | Cuánto tiempo se conservan logs y copias antes de borrarlos. |
| **Rotación de logs** | Comprimir y borrar logs antiguos para que no llenen el disco. |
| **Ventana de mantenimiento** | Franja horaria pactada con el cliente para hacer cambios. |
| **Registro de cambios (*changelog*)** | Documento donde se anota cada cambio realizado. |
| **Resumen ejecutivo** | Parte inicial del informe, breve y sin tecnicismos. |
| **Métrica / indicador** | Dato medible que resume algo (número de ataques, IPs únicas...). |
| **Línea base** | Nivel "normal" de actividad, para detectar lo que se sale de lo habitual. |
| **IoC** | *Indicator of Compromise*: dato que delata una amenaza (una IP, un hash de malware...). |

---

## Qué se mantiene y cómo se puede hacer

### Plan de mantenimiento por frecuencia

| Frecuencia | Tareas |
|---|---|
| **Diaria** (automática) | Comprobar que los servicios y honeypots están activos, revisar alertas críticas y ejecutar las copias de seguridad. |
| **Semanal** | Revisar paneles y alertas, ajustar falsos positivos, comprobar espacio en disco y revisar los logs del firewall. |
| **Mensual** | Actualizar sistema, contenedores y reglas de detección; probar una restauración de copia; revisar reglas de firewall y accesos. |
| **Trimestral** | Revisar la arquitectura y las políticas de acceso, actualizar la documentación y revisar la retención de logs. |
| **Tras un incidente** | Restaurar el honeypot afectado, analizar lo ocurrido y documentarlo. |

### Formas de hacerlo

Casi todas las tareas se pueden hacer **a mano** o **automatizarlas**. La regla práctica: lo que se repite, se automatiza; lo que requiere criterio (revisar alertas, decidir un cambio), lo hace una persona.

| Tarea | Cómo se puede hacer | Herramientas |
|---|---|---|
| **Actualizar el sistema** | Manualmente en la ventana de mantenimiento, o con actualizaciones automáticas de seguridad. | `apt`, `unattended-upgrades` |
| **Actualizar contenedores** | Descargar la nueva imagen y recrear el contenedor. | Docker Compose |
| **Actualizar reglas de detección** | Desde el propio paquete o de forma programada. | Suricata, Wazuh |
| **Copias de seguridad** | Programadas y con comprobación periódica de restauración. | `rsync`, `tar`, `vzdump` + `cron`/`systemd` (ver [Backup](backup.md)) |
| **Comprobar que todo funciona** | Una herramienta que vigile y avise cuando algo cae. | Wazuh, Grafana, comprobaciones programadas + Telegram |
| **Controlar el disco** | Rotación y retención de logs; alerta cuando se llena. | `logrotate`, alertas de Grafana |
| **Restaurar un honeypot** | Volver al snapshot limpio (ver el [ejemplo](#ejemplo-restaurar-un-honeypot-comprometido)). | Proxmox |
| **Revisar alertas** | Revisión periódica por una persona, ajustando reglas. | Wazuh, Grafana |
| **Registrar cambios** | Anotar cada cambio con fecha y motivo. | Un documento compartido o el propio repositorio |

> Antes de cualquier actualización importante, conviene hacer un **snapshot** de la VM (ver [Proxmox](proxmox.md)) para poder volver atrás.

### Ejemplo: restaurar un honeypot comprometido

Si un honeypot se compromete, el proceso general es siempre el mismo:

1. **Aislarlo:** cortar su red para que no pueda atacar a nadie.
2. **Conservar evidencias:** guardar logs y, si interesa, una copia del estado.
3. **Restaurar:** volver al snapshot limpio.
4. **Comprobar** que arranca y vuelve a registrar.
5. **Anotarlo** en el registro de cambios y, si es relevante, en el informe.

---

## Reportar al cliente

### De dónde salen los datos

Todo lo que se cuenta en el informe sale de los registros que ya se recogen. Hay varias formas de obtenerlos, y se pueden combinar:

| Forma | Cuándo conviene |
|---|---|
| **Paneles de Grafana** | Para ver tendencias y gráficos de un vistazo; también sirven como capturas para el informe. |
| **Consultas a la base de datos** | Para sacar cifras concretas (por ejemplo, las IPs más activas de la semana). |
| **Análisis de los logs** | Para detalles que no están en la base de datos, como los comandos que ejecuta un atacante. |
| **Wazuh** | Para alertas, eventos de seguridad y estado de los equipos. |

### Preguntas que un informe responde

Una buena manera de decidir qué incluir es pensar en qué querría saber el cliente:

- ¿**Cuántos ataques** recibimos y cómo evolucionan?
- ¿**Qué servicios** atacan más?
- ¿**De dónde** vienen (IPs y países)?
- ¿**Qué usuarios y contraseñas** prueban?
- ¿**Qué comandos** ejecutan cuando "entran"?
- ¿Se ha capturado algún **malware**?
- ¿Ha pasado algo **fuera de lo normal**?
- ¿**Está todo funcionando** correctamente?
- ¿**Qué debería hacer** el cliente con esta información?

### Estructura de un informe

1. **Resumen ejecutivo:** tres o cuatro líneas sobre qué ha pasado y qué se recomienda.
2. **Métricas clave:** los números principales, comparados con el periodo anterior.
3. **Principales ataques:** servicios más atacados y técnicas más frecuentes.
4. **Origen de los ataques:** IPs y países más activos.
5. **Qué intentan:** credenciales probadas, comandos y malware capturado.
6. **Incidentes destacados:** eventos fuera de lo normal y cómo se gestionaron.
7. **Estado del sistema:** disponibilidad y mantenimiento realizado.
8. **Recomendaciones:** medidas sugeridas según lo observado.

### Ejemplo de resumen ejecutivo

> *Durante la semana del 6 al 12 de octubre, el sistema registró 4.230 intentos de acceso, un 18 % más que la semana anterior. La mayoría fueron intentos automáticos de fuerza bruta contra SSH desde 312 direcciones distintas. No se detectó ningún acceso a los sistemas reales del cliente. Se recomienda desactivar el acceso por contraseña en los servidores SSH reales y revisar las credenciales por defecto.*

(Los datos de este ejemplo son inventados; solo ilustran el tono y la longitud adecuados.)

### Formas de entregarlo

| Formato | Ventajas | Inconvenientes |
|---|---|---|
| **Documento PDF** | Fácil de leer y archivar; formal. | Hay que generarlo cada vez. |
| **Panel en línea** (Grafana) | Siempre actualizado y consultable. | El cliente necesita acceso. |
| **Ambos** | Un resumen formal y un panel para el detalle. | Más trabajo de mantenimiento. |

La generación de informes automáticos en PDF desde Grafana puede requerir una edición de pago; conviene comprobarlo en su documentación. Otra opción es preparar el informe a partir de una plantilla y capturas de los paneles.

### Buenas prácticas

- **Empezar por lo importante** y dejar el detalle técnico para el final o para anexos.
- **Comparar con periodos anteriores:** un número aislado dice poco; la tendencia, mucho.
- **Interpretar los datos**, no solo mostrarlos: qué significan y qué hacer.
- **Mantener un formato constante** entre informes para que sean comparables.
- **Proteger los datos:** las direcciones IP pueden considerarse datos personales; valorar qué detalle se incluye y a quién se envía.

---

## Resumen

- Mantener un honeypot es tan importante como instalarlo: actualizar, copiar, vigilar, restaurar y revisar alertas.
- Las tareas se organizan por frecuencia; lo repetitivo se automatiza y lo que requiere criterio lo hace una persona.
- Los datos del informe salen de paneles, bases de datos, logs y del SIEM.
- Un buen informe tiene resumen ejecutivo, métricas, tendencias, incidentes y recomendaciones.

## En HoneyNet

Mantener y reportar son dos de los tres pilares del servicio de HoneyNet: el cliente no paga solo por la instalación, sino por tener el entorno funcionando y por entender lo que ocurre en su red.

## Referencias

- Wazuh: <https://documentation.wazuh.com/>
- Grafana: <https://grafana.com/docs/>
- Proxmox, copias y restauración: <https://pve.proxmox.com/wiki/Backup_and_Restore>

[← Teoría](README.md)
