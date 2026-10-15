# Stack tecnológico

> [!NOTE]
> **Borrador.** Construido a partir de la idea principal del proyecto.

La columna **Rol** indica a quién pertenece cada herramienta:

- **HoneyNet:** forma parte del servicio que se implanta en el cliente.
- **Cliente:** pertenece a la red del cliente (simulada en el laboratorio).
- **Laboratorio:** solo existe para simular el escenario o para proteger el propio proyecto.

| Capa | Herramienta | Para qué la usamos | Rol |
|---|---|---|---|
| Virtualización | **Proxmox VE** | Hipervisor sobre el que corre todo el laboratorio. | Laboratorio |
| Atacante de pruebas | **Kali Linux** | Generar ataques controlados. | Laboratorio |
| Segmentación | **VLANs 10, 20, 30, 40 y 99** | DMZ, servidores, gestión, usuarios y exterior. | HoneyNet |
| Red y seguridad perimetral | **pfSense** | Firewall y router entre VLANs. | HoneyNet |
| Detección | **Suricata** (reglas ET Open) | IDS integrado en el firewall. | HoneyNet |
| Contenedores | **Docker + Compose** | Despliegue de los honeypots en la DMZ. | HoneyNet |
| Honeypots | **Cowrie, Dionaea, DVWA** | Señuelos SSH/Telnet, malware y web. | HoneyNet |
| SIEM | **Wazuh** | Centralización y correlación de logs. | HoneyNet |
| Visualización | **Grafana** | Dashboards y mapa de atacantes. | HoneyNet |
| Alertas y respuesta | **Telegram, Python, AbuseIPDB** | Avisos y script de respuesta automática. | HoneyNet |
| Informes al cliente | **Grafana + informe periódico** | Resumen de ataques, orígenes y estado del sistema (formato por definir). | HoneyNet |
| Acceso de administración | **NetBird** *(propuesto)* | Acceso remoto cifrado sin abrir puertos de gestión. | HoneyNet |
| Firewall en Linux | **nftables** *(propuesto)* | Filtrado y DNAT en las VMs Linux. | HoneyNet |
| Base de datos | **MySQL / MariaDB** | Guardar los datos de los ataques capturados. | HoneyNet |
| Copias de seguridad del servicio | **rsync + tar + cron/systemd** | Copias de la base de datos, los logs y las configuraciones. | HoneyNet |
| Servidores internos | **Bind9, isc-dhcp-server** | DNS y DHCP de la red del cliente. | Cliente |
| Equipos de empleados | **VM Windows/Linux** | Representan a la plantilla del cliente. | Cliente |
| Copias del proyecto | **vzdump (Proxmox) + GitHub** | Proteger el propio proyecto: VMs, configuraciones y documentación. | Laboratorio |

[← Introducción](README.md)
