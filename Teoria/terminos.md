# Términos

Glosario general de los conceptos que aparecen en la teoría y en el proyecto, agrupado por tema.

## Virtualización y contenedores

| Término | Significado |
|---|---|
| **Hipervisor** | Software que crea y gestiona máquinas virtuales. Tipo 1: sobre el hardware (Proxmox). Tipo 2: sobre un sistema operativo (VirtualBox). |
| **Node (nodo)** | Equipo físico donde corre el hipervisor. |
| **VM** | Máquina virtual: ordenador completo simulado. |
| **KVM / QEMU** | Tecnología de virtualización de Linux / emulador de hardware que la acompaña. |
| **LXC** | Contenedor de Linux que comparte el kernel del anfitrión. |
| **Bridge (`vmbr`)** | Switch virtual al que se conectan las VMs. |
| **Snapshot** | Foto del estado de una VM que permite volver atrás. |
| **Template** | VM maestra que se clona para crear otras. |
| **Storage** | Lugar donde se guardan discos, ISOs y copias. |
| **Imagen (Docker)** | Plantilla de solo lectura de un contenedor. |
| **Contenedor** | Imagen en ejecución. |
| **Volumen** | Almacenamiento persistente fuera del contenedor. |
| **Docker Compose** | Definición de varios contenedores en un archivo. |
| **Dockerfile** | Receta para construir una imagen. |

## Redes

| Término | Significado |
|---|---|
| **VLAN** | Red local virtual que segmenta el tráfico a nivel 2. |
| **802.1Q** | Estándar de etiquetado de VLANs. |
| **Trunk / Access** | Puerto que transporta varias VLANs / puerto que pertenece a una sola. |
| **Router** | Dispositivo que conecta redes distintas. |
| **Gateway (puerta de enlace)** | Equipo por el que salen los paquetes hacia otras redes. |
| **NAT** | Traducción de direcciones IP privadas a una pública. |
| **DNAT / Port forward** | Redirección de un puerto entrante hacia una máquina interna. |
| **Masquerade** | Forma de NAT saliente que usa la IP de la interfaz de salida. |
| **DMZ** | Zona expuesta, separada de la red interna, donde se ponen los servicios accesibles desde fuera. |
| **WAN / LAN** | Red externa / red local. |
| **Subred / CIDR** | Parte de una red / notación `10.0.0.0/24`. |
| **DNS** | Sistema que traduce nombres en IPs. |
| **DHCP** | Protocolo que asigna configuración de red automáticamente. |
| **Broadcast** | Mensaje dirigido a todos los equipos de una red. |
| **Overlay** | Red virtual construida sobre otra red. |
| **VPN** | Red privada virtual: conexión cifrada entre redes o equipos. |
| **Túnel VPN** | Conexión cifrada entre dos puntos que atraviesa Internet como un cable privado. |
| **VPS** | Servidor virtual alquilado en la nube, con IP pública. |
| **PVID** | VLAN por defecto de un puerto: la que se asigna a las tramas sin etiqueta que entran por él. |
| **Router-on-a-stick** | Un router o firewall con una sola tarjeta de red que atiende varias VLANs por un trunk. |
| **DHCP relay** | Reenvía las peticiones DHCP de una VLAN a un servidor situado en otra. |
| **WireGuard** | Protocolo de VPN moderno y ligero. |
| **ZTNA** | Acceso de confianza cero: basado en identidad y mínimo privilegio. |

## Seguridad

| Término | Significado |
|---|---|
| **Firewall** | Sistema que filtra el tráfico según reglas. |
| **UFW** | Herramienta sencilla para configurar el firewall de Linux. |
| **Stateful** | Firewall que recuerda el estado de las conexiones. |
| **iptables / nftables** | Firewalls del kernel de Linux (nftables es el sucesor). |
| **pfSense** | Sistema de firewall/router de código abierto basado en FreeBSD. |
| **IDS / IPS** | Sistema de detección / de prevención de intrusiones. |
| **Suricata** | Motor IDS/IPS de código abierto. |
| **Firma (regla IDS)** | Patrón que identifica un ataque conocido. |
| **Defensa en profundidad** | Uso de varias capas de protección independientes. |
| **Mínimo privilegio** | Dar solo los permisos estrictamente necesarios. |
| **Hardening** | Endurecimiento de la configuración de un sistema. |
| **Fuerza bruta** | Probar muchas credenciales hasta acertar. |
| **Malware** | Software malicioso. |
| **SQL injection** | Ataque que inserta código SQL en una aplicación. |
| **XSS** | Inyección de scripts en páginas web. |
| **Falso positivo** | Alerta que salta sin que haya un ataque real. |
| **MFA / SSO** | Autenticación multifactor / inicio de sesión único. |
| **Threat intelligence** | Información sobre amenazas y atacantes. |

## Honeypots y monitorización

| Término | Significado |
|---|---|
| **Honeypot** | Sistema señuelo diseñado para ser atacado. |
| **Honeynet** | Red de honeypots con herramientas de monitorización. |
| **Honeytoken** | Dato señuelo que delata un acceso indebido. |
| **Interacción baja / media / alta** | Nivel de realismo de un honeypot. |
| **Cowrie** | Honeypot SSH/Telnet. |
| **Dionaea** | Honeypot que captura malware. |
| **DVWA** | Aplicación web intencionadamente vulnerable. |
| **Log** | Registro de eventos de un sistema. |
| **SIEM** | Plataforma que centraliza y correlaciona logs de seguridad. |
| **Wazuh** | Plataforma SIEM/XDR de código abierto. |
| **Agente** | Programa que envía datos de un equipo a un servidor central. |
| **Correlación** | Relacionar varios eventos para detectar un patrón. |
| **Grafana** | Herramienta de visualización y paneles. |
| **Dashboard** | Panel visual con gráficos y datos. |
| **AbuseIPDB** | Base de datos de reputación de direcciones IP. |

## Administración y copias de seguridad

| Término | Significado |
|---|---|
| **Backup** | Copia de seguridad de los datos. |
| **Copia completa / incremental / diferencial** | Todo / cambios desde la última copia / cambios desde la última completa. |
| **Regla 3-2-1** | 3 copias, 2 soportes, 1 fuera del sitio. |
| **RPO / RTO** | Pérdida de datos aceptable / tiempo de recuperación aceptable. |
| **Retención** | Tiempo que se conservan las copias. |
| **tar** | Herramienta para empaquetar y comprimir ficheros. |
| **rsync** | Herramienta para sincronizar ficheros, local o por SSH. |
| **cron / crontab** | Programador clásico de tareas en Linux. |
| **systemd timer** | Programador de tareas de systemd, gestionado con `systemctl`. |
| **SSH** | Protocolo de acceso remoto cifrado. |
| **Clave SSH** | Par de claves que permite entrar sin contraseña. |
| **Hash (sha256)** | Huella de un fichero, útil para comprobar su integridad. |
| **Informe / reporte** | Documento periódico que resume al cliente lo ocurrido y el estado del sistema. |
| **Resumen ejecutivo** | Parte inicial de un informe, breve y sin tecnicismos. |
| **Rotación de logs** | Comprimir y borrar logs antiguos para que no llenen el disco. |
| **MySQL / MariaDB** | Gestores de bases de datos relacionales. |
| **nginx** | Servidor web ligero y muy usado. |
| **PHP-FPM** | Proceso que ejecuta código PHP para que nginx pueda servir páginas dinámicas. |
| **Proxy inverso** | Servidor que recibe las peticiones y las reenvía a otro servidor. |
| **PDO** | Biblioteca de PHP para acceder a bases de datos con consultas preparadas. |
| **Consulta preparada** | Consulta en la que la estructura y los datos van separados, contra la inyección SQL. |
| **XSS** | Inyección de scripts en páginas web. |
| **.env** | Archivo con credenciales, fuera de la carpeta pública y de Git. |
| **honeypot.lan** | Dominio interno de la red del proyecto, resuelto por Bind9. |
| **Certificado autofirmado** | Certificado TLS generado por uno mismo; cifra pero los navegadores no confían en él. |
| **Endurecimiento** | Ajustar la configuración de un servicio para reducir riesgos. |
| **SQL** | Lenguaje de consulta de bases de datos. |

[← Teoría](README.md)
