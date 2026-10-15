# pfSense y Suricata

## En simple

### Qué es pfSense

**pfSense** es un sistema operativo **gratuito y de código abierto** (basado en FreeBSD) diseñado para convertir un equipo, físico o virtual, en un **firewall y router**. Se maneja desde una interfaz web y reúne en un solo sistema muchas funciones que de otra forma requerirían varios equipos.

> **Analogía:** es el portero y la aduana de una red. Decide quién entra, quién sale y por dónde, y lleva un registro de todo.

Lo que ofrece:

- **Firewall con seguimiento de estado** (*stateful*): recuerda las conexiones abiertas y permite automáticamente las respuestas.
- **Router y NAT**: conecta redes entre sí y traduce direcciones.
- **Servicios de red**: DHCP, DNS (Unbound), etc.
- **VPN**: OpenVPN, WireGuard, IPsec.
- **Paquetes adicionales**: Suricata o Snort (IDS/IPS), pfBlockerNG (listas de bloqueo), entre otros.

### Qué es Suricata

**Suricata** es un motor de detección de intrusiones de código abierto. Analiza el tráfico de red buscando patrones conocidos de ataque (firmas) y genera alertas.

- **IDS** (*Intrusion Detection System*): **detecta** y avisa, pero no corta el tráfico.
- **IPS** (*Intrusion Prevention System*): además **bloquea** lo malicioso.

> **Analogía:** el firewall es el portero que mira quién tiene permiso de entrar; el IDS es el vigilante de seguridad que observa qué hace la gente dentro y da la alarma si alguien se comporta de forma sospechosa.

---

## Conceptos clave

- **Interfaz WAN / LAN / OPT**: WAN es la que mira hacia la red externa; LAN, la red interna; las OPT son interfaces adicionales (por ejemplo, una por VLAN).
- **Regla de firewall**: condición (origen, destino, protocolo, puerto) más acción (*Pass*, *Block* o *Reject*).
- **Orden de reglas**: se evalúan **de arriba abajo en cada interfaz y se aplica la primera que coincide**.
- **Denegación por defecto**: lo que no está permitido explícitamente en la entrada de una interfaz, se bloquea. En la WAN todo el tráfico entrante está bloqueado por defecto; en la LAN existe por defecto una regla que permite todo, y suele endurecerse.
- **Alias**: nombre que agrupa IPs, redes o puertos, para no repetirlos en cada regla.
- **NAT saliente (*outbound*)**: oculta las IPs privadas detrás de la IP de la WAN para salir a Internet.
- **Port forward (DNAT)**: redirige un puerto de la WAN hacia una máquina interna.
- **Estado (*state*)**: entrada en la tabla de conexiones que permite el tráfico de vuelta.
- **Reglas ET Open**: conjunto gratuito de firmas *Emerging Threats* para Suricata.

---

## En técnico

### Instalación (resumen)

1. Descargar la ISO desde la web oficial y crear una VM (en Proxmox: tipo Other/FreeBSD, disco de unas decenas de GB, **al menos dos interfaces de red**: WAN y LAN).
2. Instalar con el asistente (opciones por defecto: *Auto (ZFS)* o *Auto (UFS)*).
3. En la consola, asignar las interfaces (`vtnet0` = WAN, `vtnet1` = LAN) y poner la IP de la LAN.
4. Acceder por navegador a la IP de la LAN. Por defecto: `https://192.168.1.1`, usuario `admin`, contraseña `pfsense`. **Cambiar la contraseña** en el primer acceso.

> El asistente inicial de la web permite fijar nombre, DNS, zona horaria y contraseña.

### Configuración de VLANs

*Interfaces → Assignments → VLANs → Add*: se elige la interfaz padre, el **VLAN Tag** y una descripción. Después se asigna cada VLAN como una interfaz nueva (*OPT1*, *OPT2*...), se le da IP (por ejemplo `10.0.10.1/24`) y se activa. Cada interfaz tendrá su propia pestaña de reglas.

### Reglas de firewall

*Firewall → Rules → (interfaz)*. Ejemplo conceptual para una VLAN de servidores:

| Acción | Protocolo | Origen | Destino | Puerto | Descripción |
|---|---|---|---|---|---|
| Pass | UDP/TCP | VLAN Servidores | Cualquiera | 53 | Permitir DNS |
| Pass | TCP | VLAN Servidores | Cualquiera | 80, 443 | Permitir actualizaciones |
| Block | Cualquiera | VLAN Servidores | Otras VLANs internas | * | Bloquear acceso lateral |
| Block | Cualquiera | Cualquiera | Cualquiera | * | Bloqueo final y registro |

Buenas prácticas:

- Poner **primero lo más específico** y al final el bloqueo general.
- Usar **alias** para listas de redes y puertos.
- Activar el **registro** (*Log*) en las reglas de bloqueo importantes.
- Documentar cada regla en el campo *Description*.

### NAT

- **Port forward** (*Firewall → NAT → Port Forward*): redirigir, por ejemplo, el puerto 2222 de la WAN a un servidor de la DMZ. pfSense crea la regla de firewall asociada.
- **Outbound NAT** (*Firewall → NAT → Outbound*): por defecto en modo automático; enmascara el tráfico saliente de las redes internas.

### Suricata en pfSense

Se instala desde *System → Package Manager → Available Packages → Suricata*. Después, en *Services → Suricata*:

1. **Global Settings**: activar la descarga de las reglas **ET Open** (y actualizarlas).
2. **Interfaces → Add**: elegir la interfaz a vigilar (normalmente la WAN o la DMZ).
3. Habilitar las categorías de reglas deseadas.
4. **Modo de funcionamiento**:
   - *Legacy mode*: solo detecta y genera alertas (IDS).
   - *Inline mode*: puede descartar paquetes (IPS); requiere compatibilidad con la tarjeta de red.
5. Activar, si se quiere, **Block Offenders** para bloquear automáticamente las IPs que disparan alertas.
6. Revisar las alertas en *Services → Suricata → Alerts*.

> Un IPS mal ajustado puede bloquear tráfico legítimo (falsos positivos). Conviene empezar en modo IDS, revisar las alertas y activar el bloqueo después.

### Automatización

pfSense CE no incluye una API oficial para gestionar el firewall desde scripts; existen paquetes de terceros que añaden una API REST. Si se quiere bloquear IPs desde un script, hay que valorar esa opción o alternativas como ejecutar comandos por SSH.

### Alternativas

- **OPNsense**: proyecto parecido (nacido como bifurcación de pfSense), también basado en FreeBSD.
- **Firewall en Linux** con nftables (ver [nftables](nftables.md)).

---

## Resumen

- pfSense es un firewall/router con interfaz web; las reglas se evalúan de arriba abajo en cada interfaz y gana la primera que coincide.
- Soporta VLANs, NAT, DHCP, DNS y VPN.
- Suricata añade detección (IDS) y, opcionalmente, prevención (IPS) de intrusiones con reglas como ET Open.

## En HoneyNet

pfSense con Suricata actúa como firewall, router entre VLANs e IDS en una sola VM. Las reglas implementan la tabla de tráfico del [diagrama de red](../Introducci%C3%B3n/diagrama-de-red.md).

## Referencias

- Documentación oficial: <https://docs.netgate.com/pfsense/en/latest/>
- Suricata: <https://suricata.io/>
- Reglas Emerging Threats: <https://rules.emergingthreats.net/>

[← Teoría](README.md)
