# Stack tecnológico

## Introducción

Tabla única con todas las herramientas del proyecto: qué hace cada una, en qué fase se instala y dónde está su documentación oficial. Funciona como índice rápido del laboratorio.

---

## Resumen por capas

### Virtualización e infraestructura

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **Proxmox VE** | Hipervisor bare-metal instalado en los dos PCs (dos nodos independientes, sin clúster). | F1 | [pve.proxmox.com/pve-docs](https://pve.proxmox.com/pve-docs/) |
| **Contenedores LXC** | Servidores ligeros de Proxmox: DNS y DHCP, base de datos, copias y web. | F3 | [pve.proxmox.com](https://pve.proxmox.com/pve-docs/chapter-pct.html) |
| **Docker + Compose** | Empaqueta y despliega honeypots, Wazuh y Grafana. | F4–F6 | [docs.docker.com](https://docs.docker.com/) |

### Red y seguridad perimetral

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **pfSense** | Firewall y router entre todas las VLANs, en una VM del PC 1. | F2 | [docs.netgate.com](https://docs.netgate.com/pfsense/en/latest/) |
| **Suricata** | IDS dentro de pfSense; alerta con firmas ET Open. | F2 | [suricata.io](https://suricata.io/documentation/) |
| **UFW** | Cortafuegos de cada servidor, con mínimo privilegio. | F3 | [help.ubuntu.com](https://help.ubuntu.com/community/UFW) |

### Servicios internos y web

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **Bind9** | Servidor DNS interno (`ct-dns-dhcp`). | F3 | [bind9.readthedocs.io](https://bind9.readthedocs.io/) |
| **DHCP (Kea o isc-dhcp-server)** | Reparte IPs en la VLAN de Usuarios, con relay en pfSense. | F3 | [isc.org/kea](https://www.isc.org/kea/) |
| **MariaDB** | Base de datos del contenido de la web (`ct-db`). | F3 | [mariadb.org](https://mariadb.org/documentation/) |
| **rsync + cron** | Copias de seguridad automáticas nocturnas (`ct-backup`). | F3 | [rsync.samba.org](https://rsync.samba.org/documentation.html) |
| **nginx + PHP-FPM** | Servidor web de la empresa y su lógica en PHP (`ct-web`, VLAN 60). | F3 | [nginx.org/en/docs](https://nginx.org/en/docs/) |
| **Bootstrap** | Frontend responsive de la web. | F3 | [getbootstrap.com](https://getbootstrap.com/docs/5.3/) |

### Honeypots

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **Cowrie** | Honeypot SSH/Telnet con grabación de sesiones. | F4 | [github.com/cowrie/cowrie](https://github.com/cowrie/cowrie) |
| **Dionaea** | Honeypot multi-protocolo para capturar malware. | F4 | [github.com/DinoTools/dionaea](https://github.com/DinoTools/dionaea) |
| **DVWA** | Aplicación web vulnerable para registrar ataques web. | F4 | [github.com/digininja/DVWA](https://github.com/digininja/DVWA) |

### Monitorización y respuesta

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **Wazuh (completo)** | SIEM: centraliza logs y aplica reglas de correlación; guarda los ataques. | F5 | [documentation.wazuh.com](https://documentation.wazuh.com/current/index.html) |
| **Grafana** | Dashboards, mapa y tabla de logs a partir de Wazuh; se incrustan en `/admin`. | F6 | [grafana.com/docs](https://grafana.com/docs/grafana/latest/) |
| **Telegram Bot API** | Alertas al móvil, enviadas desde Grafana. | F6 | [core.telegram.org/bots/api](https://core.telegram.org/bots/api) |
| **Python + AbuseIPDB** | *Extra opcional:* script de respuesta y bloqueo automático de IPs en pfSense. | Opcional | [docs.abuseipdb.com](https://docs.abuseipdb.com/) |

### Hardware y acceso remoto

| Elemento | Función | Fase | Documentación |
|---|---|---|---|
| **PC 1 (Proxmox)** | Aloja pfSense (VM), honeypots, servidores, web y cliente. | F1 | — |
| **PC 2 (Proxmox)** | Aloja el SIEM (Wazuh y Grafana) y Kali. | F1 | — |
| **Switch HP 1810-24** | Conecta los PCs y transporta las VLANs. | F2 | — |
| **NetBird** | VPN para trabajar en el laboratorio desde casa. | F1 | [docs.netbird.io](https://docs.netbird.io/) |
| **Git + GitHub** | Control de versiones del código y de la documentación. | F1 | [docs.github.com](https://docs.github.com/) |

> Detalle del hardware en [Infraestructura física](infraestructura-fisica.md). El firewall Juniper NetScreen-25 está aparcado y no forma parte del stack.

### Máquinas auxiliares

| Herramienta | Función | Fase | Documentación |
|---|---|---|---|
| **Kali Linux** | Máquina atacante en la VLAN 50 (PC 2), encendida solo en pruebas. | F2 en adelante | [kali.org/docs](https://www.kali.org/docs/) |
| **VM cliente (Debian)** | Equipo de empleado simulado en la VLAN de Usuarios (PC 1). | F3 | — |

---

> Para ver dónde vive cada herramienta dentro de la red, consulta el [Diagrama de red](diagrama-de-red.md). Para entender por qué cada pieza está ahí, consulta la [Teoría](../Teoria/README.md).

## Próximos pasos

- Añadir la versión exacta de cada herramienta cuando se instale.
- Anotar aquí cualquier herramienta nueva que se incorpore durante el proyecto.

[← Volver a Introducción](README.md)
