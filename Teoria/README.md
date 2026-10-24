# Teoría

Base teórica de cada tecnología que aparece en el proyecto. Cada página está pensada para leerse de dos formas:

- **En simple**: qué es y para qué sirve, con una analogía, en lenguaje sencillo pero formal.
- **En técnico**: comandos, ficheros de configuración y ejemplos.

Las páginas explican cada tema **en general**; al final de cada una hay un apartado corto, **En HoneyNet**, que indica dónde se usa en el proyecto.

## Índice

| # | Página | Contenido |
|---|---|---|
| 1 | [Proxmox](proxmox.md) | Virtualización, VMs y contenedores, almacenamiento, redes, snapshots y templates |
| 2 | [Docker](docker.md) | Imágenes, contenedores, volúmenes, redes y Docker Compose |
| 3 | [Backup](backup.md) | Teoría de copias de seguridad, `tar`, `rsync`, `cron` y `systemd` timers |
| 4 | [VLANs](vlans.md) | Segmentación de redes, 802.1Q, trunk y access, PVID, DHCP relay y enrutado entre VLANs |
| 5 | [pfSense y Suricata](pfsense.md) | Firewall/router, reglas, NAT, VLANs, router-on-a-stick e IDS/IPS |
| 6 | [UFW y SSH con claves](ufw-y-ssh.md) | Firewall de cada servidor, mínimo privilegio y acceso remoto seguro |
| 7 | [nftables](nftables.md) | Firewall de Linux, tablas, cadenas, NAT, sets y logging |
| 8 | [NetBird](netbird.md) | Red privada superpuesta (WireGuard) y acceso remoto |
| 9 | [VPS y túneles VPN](vps-y-tuneles.md) | Honeypots en la nube y túnel cifrado (ampliación futura) |
| 10 | [Honeypots](honeypots.md) | Qué son, tipos, Cowrie, Dionaea y DVWA |
| 11 | [Servicios internos](servicios-internos.md) | DNS (Bind9), DHCP y MariaDB/MySQL |
| 12 | [Servidor web (nginx)](nginx.md) | Servidor web, PHP-FPM, HTTPS, proxy inverso y endurecimiento |
| 13 | [Seguridad web (PHP + MariaDB)](seguridad-web.md) | Inyección SQL, XSS, contraseñas, sesiones y credenciales |
| 14 | [Monitorización y respuesta](wazuh-grafana.md) | SIEM, Wazuh, Grafana, alertas y scripts de respuesta |
| 15 | [Mantenimiento y reportes](mantenimiento-y-reportes.md) | Plan de mantenimiento, restauración e informes al cliente |
| 16 | [Términos](terminos.md) | Glosario general |

## Convención de cada página

1. En simple
2. Conceptos clave
3. En técnico
4. Resumen y referencias
5. En HoneyNet *(opcional)*

[← Volver al inicio](../README.md)
