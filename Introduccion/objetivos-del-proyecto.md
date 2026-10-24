# Objetivos del proyecto

> [!NOTE]
> **Borrador.** Redactado a partir de la idea principal y de la arquitectura; ajustadlo a la memoria final.

## Objetivo general

Demostrar cómo **HoneyNet**, una empresa de ciberseguridad defensiva, implanta, mantiene y reporta honeypots en una red de empresa. Para ello se monta un laboratorio con hardware real que simula esa red, y sobre ella se despliega la solución completa, que cierra el ciclo:

```
detectar → analizar → responder → visualizar
```

## Objetivos específicos

### A. Servicio de HoneyNet

**Implantación**

1. **Segmentar la red** en seis VLANs internas más la WAN (DMZ, servidores, gestión, usuarios, atacante y web) con un switch gestionable.
2. **Instalar un firewall con detección de intrusiones** (pfSense + Suricata) con una sola tarjeta de red, con reglas de denegar por defecto entre zonas.
3. **Desplegar los honeypots** (Cowrie, Dionaea y DVWA) en la DMZ para capturar ataques SSH/Telnet, malware y explotación web.

**Monitorización y respuesta**

4. **Centralizar los logs** en un SIEM (Wazuh) con un mínimo de reglas personalizadas.
5. **Visualizar** los ataques en Grafana (totales, ataques por hora, IPs, contraseñas, mapa mundial, tipos de ataque y comandos).
6. **Alertar** a Telegram desde Grafana, sin escribir código.
7. **Mostrar los ataques en la web**, en un panel de administración con login que incrusta los paneles de Grafana.
8. *(Extra opcional)* **Bloquear automáticamente** las IPs atacantes en el firewall.

**Mantenimiento y reporte**

9. **Definir el mantenimiento:** actualizaciones, copias de seguridad nocturnas, restauración de honeypots y revisión de alertas.
10. **Analizar los ataques capturados** y preparar un reporte para el cliente, anonimizando las direcciones IP.

### B. Empresa simulada y laboratorio

11. **Montar el laboratorio** con dos PCs con Proxmox y un switch HP 1810-24, con los recursos limitados del centro.
12. **Reproducir la red de la empresa:** DNS (Bind9), DHCP, base de datos (MariaDB), copias de seguridad (rsync), servidor web (nginx + PHP) y un equipo de empleado.
13. **Desarrollar la web de la empresa** con vídeo, sectores, planes, formulario de contacto y zona de administración.
14. **Proteger los servidores** con UFW, SSH con claves, consultas preparadas, credenciales ocultas y usuarios de base de datos con mínimos privilegios.
15. **Preparar un atacante de pruebas** (Kali Linux) en una VLAN aislada, para validar el sistema.
16. **Administrar el laboratorio a distancia** con NetBird.

### C. Documentación

17. **Documentar** todo el proceso con control de versiones, y elaborar la memoria y la presentación.

[← Volver a Introducción](README.md)
