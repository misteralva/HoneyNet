# HoneyNet

**Hecho por Álex Naranjo Blázquez y David Álvarez Fernández**

Repositorio de documentación de nuestro proyecto de segundo curso del ciclo de **Administración de Sistemas Informáticos en Red (ASIR)**, con perfil en **ciberseguridad**.

## ¿De qué trata?

**HoneyNet** es una empresa (ficticia) que **vende un servicio de honeypots**: los implanta en la red de otras empresas, los mantiene y les enseña lo que ocurre. Este proyecto documenta cómo lo montamos en una **red de empresa simulada**: dos PCs con Proxmox, un switch con VLANs, pfSense con Suricata, tres honeypots, Wazuh y Grafana, una web con panel de administración y alertas por Telegram. Empieza por [La idea principal](Introducci%C3%B3n/ideas-principales.md).

## Índice

### Introducción al proyecto

| Página | Contenido |
|---|---|
| [La idea principal](Introducci%C3%B3n/ideas-principales.md) | Explicación completa y sencilla del proyecto |
| [Objetivos](Introducci%C3%B3n/objetivos-del-proyecto.md) | Objetivo general y objetivos específicos |
| [Infraestructura física](Introducci%C3%B3n/infraestructura-fisica.md) | El hardware real y las decisiones tomadas |
| [Diagrama de red](Introducci%C3%B3n/diagrama-de-red.md) | Arquitectura, VLANs y reglas de tráfico |
| [Stack tecnológico](Introducci%C3%B3n/stack-tecnologico.md) | Todas las herramientas y su documentación |
| [Arquitectura del software](Introducci%C3%B3n/arquitectura-del-software/README.md) | Requisitos, componentes, base de datos, interfaz, red y seguridad |
| [Parte web](Introducci%C3%B3n/parte-web/diseno-y-plan-de-produccion.md) | Diseño y plan de producción de la web de la empresa |

### Teoría

Base teórica general de cada tecnología, con una parte **en simple** y otra **en técnico**. Ver el [índice de Teoría](Teoria/README.md).

| Página | Contenido |
|---|---|
| [Proxmox](Teoria/proxmox.md) | Virtualización, VMs y contenedores, redes, snapshots y templates |
| [Docker](Teoria/docker.md) | Imágenes, contenedores, volúmenes, redes y Compose |
| [Backup](Teoria/backup.md) | Teoría de copias, `tar`, `rsync`, `cron` y `systemd` timers |
| [VLANs](Teoria/vlans.md) | Segmentación de redes, 802.1Q, trunk y access |
| [pfSense y Suricata](Teoria/pfsense.md) | Firewall/router e IDS/IPS |
| [UFW y SSH con claves](Teoria/ufw-y-ssh.md) | Firewall de cada servidor y acceso remoto seguro |
| [nftables](Teoria/nftables.md) | Firewall de Linux |
| [NetBird](Teoria/netbird.md) | Red privada superpuesta y acceso remoto |
| [VPS y túneles VPN](Teoria/vps-y-tuneles.md) | Honeypots en la nube y túnel cifrado (ampliación futura) |
| [Honeypots](Teoria/honeypots.md) | Cowrie, Dionaea y DVWA |
| [Servicios internos](Teoria/servicios-internos.md) | DNS, DHCP y MariaDB |
| [Servidor web (nginx)](Teoria/nginx.md) | Servidor web, PHP-FPM, HTTPS y proxy inverso |
| [Seguridad web (PHP + MariaDB)](Teoria/seguridad-web.md) | Inyección SQL, XSS, contraseñas y credenciales |
| [Monitorización y respuesta](Teoria/wazuh-grafana.md) | Wazuh, Grafana y alertas |
| [Mantenimiento y reportes](Teoria/mantenimiento-y-reportes.md) | Mantenimiento del entorno e informes al cliente |
| [Términos](Teoria/terminos.md) | Glosario general |

## Estructura del repositorio

```
HoneyNet/
├── README.md
├── assets/            # imágenes y diagramas
├── Introducción/      
└── Teoria/            # teoría general de cada tecnología (simple + técnico)
```
