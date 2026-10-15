# Objetivos del proyecto

> [!NOTE]
> **Borrador.** Redactado a partir de la idea principal del proyecto; ajustadlo a la memoria final.

## Objetivo general

Demostrar cómo **HoneyNet**, una empresa dedicada a la ciberseguridad defensiva, **implanta, mantiene y reporta honeypots** en la red de un cliente. Para ello se monta un laboratorio que simula la red del cliente, y sobre ella se despliega la solución completa, que recibe ataques reales y cierra el ciclo:

```
detectar → analizar → responder → visualizar
```

## Objetivos específicos

### A. Servicio de HoneyNet

**Implantación**

1. **Segmentar la red del cliente** en zonas aisladas (VLANs): exterior, DMZ, servidores, gestión y usuarios, aplicando defensa en profundidad.
2. **Instalar un firewall con detección de intrusiones** que controle el tráfico entre zonas.
3. **Desplegar los honeypots** (Cowrie, Dionaea y DVWA) en la DMZ para capturar ataques SSH/Telnet, malware y explotación web.

**Monitorización y automatización**

4. **Centralizar los logs** en un SIEM (Wazuh) y crear reglas de correlación.
5. **Montar una base de datos propia** (MySQL) donde guardar los datos de los ataques.
6. **Visualizar y alertar:** paneles en Grafana, mapa de atacantes y avisos por Telegram.
7. **Automatizar la respuesta** con un script que enriquezca las IPs (AbuseIPDB), las guarde en la base de datos y, si da tiempo, las bloquee en el firewall.
8. **Administrar a distancia** la infraestructura de forma segura con NetBird, sin exponer puertos de gestión.

**Mantenimiento y reporte**

9. **Definir el mantenimiento:** actualizaciones, restauración de honeypots y revisión de alertas.
10. **Automatizar las copias de seguridad** de la base de datos, los logs y las configuraciones de la solución.
11. **Generar un reporte completo para el cliente** con los ataques recibidos, su origen, las técnicas usadas y el estado del sistema.

**Exposición controlada**

12. **Exponer el sistema a Internet de forma controlada**, verificando que ningún atacante alcanza las zonas internas.

### B. Laboratorio de simulación

13. **Montar el laboratorio** sobre un único equipo con Proxmox.
14. **Reproducir la red del cliente:** DNS, DHCP y equipos de empleados.
15. **Preparar un atacante de pruebas** (Kali Linux) para validar el sistema antes de exponerlo.
16. **Proteger el propio proyecto** con copias de seguridad de las máquinas virtuales, las configuraciones y la documentación, para no perder el trabajo.

### C. Documentación

17. **Documentar** todo el proceso, y elaborar la memoria y la presentación.

[← Introducción](README.md)
