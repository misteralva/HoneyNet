# HoneyNet

**Hecho por Álex Naranjo Blázquez y David Álvarez Fernández**

Repositorio de documentación de nuestro proyecto de segundo curso del ciclo de **Administración de Sistemas Informáticos en Red (ASIR)**, con perfil en **ciberseguridad**.

## ¿De qué trata?

**HoneyNet** es una empresa (ficticia) que **implanta honeypots en otras empresas, los mantiene y les entrega informes** de los ataques recibidos. Este proyecto documenta cómo implantamos el sistema en la red de un cliente: montar la red, desplegar los honeypots, recoger y analizar los logs, y automatizarlo todo. Empieza por [La idea principal](Introducci%C3%B3n/ideas-principales.md).

> **Estado actual del proyecto:** en desarrollo activo
> **Fase:** investigación y estructuración técnica inicial

En este repositorio se incluirá la memoria del proyecto, junto con todos los aspectos y acciones que se han ido tomando durante su desarrollo.

## Índice

### Introducción al proyecto

| Página | Contenido |
|---|---|
| [La idea principal](Introducci%C3%B3n/ideas-principales.md) | Explicación completa y sencilla del proyecto |
| [Objetivos del proyecto](Introducci%C3%B3n/objetivos-del-proyecto.md) | Objetivo general y objetivos específicos |
| [Stack tecnológico](Introducci%C3%B3n/stack-tecnologico.md) | Herramientas que utilizamos y para qué |
| [Diagrama de red](Introducci%C3%B3n/diagrama-de-red.md) | Arquitectura, VLANs y reglas de tráfico |

### Teoría

Base teórica general de cada tecnología, con una parte **en simple** y otra **en técnico**. Ver el [índice de Teoría](Teoria/README.md).

| Página | Contenido |
|---|---|
| [Proxmox](Teoria/proxmox.md) | Virtualización, VMs y contenedores, redes, snapshots y templates |
| [Docker](Teoria/docker.md) | Imágenes, contenedores, volúmenes, redes y Compose |
| [Backup](Teoria/backup.md) | Teoría de copias, `tar`, `rsync`, `cron` y `systemd` timers |
| [VLANs](Teoria/vlans.md) | Segmentación de redes y 802.1Q |
| [pfSense y Suricata](Teoria/pfsense.md) | Firewall/router e IDS/IPS |
| [nftables](Teoria/nftables.md) | Firewall de Linux |
| [NetBird](Teoria/netbird.md) | Red privada superpuesta y acceso remoto |
| [Honeypots](Teoria/honeypots.md) | Cowrie, Dionaea y DVWA |
| [Servicios internos](Teoria/servicios-internos.md) | DNS, DHCP y MySQL |
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
