# Proxmox

## En simple

### El problema que resuelve

Un programa como VirtualBox se instala **encima** de un sistema operativo (Windows, Linux...). Ese sistema ya consume memoria y procesador por su cuenta, y el programa de virtualización reparte entre las máquinas virtuales solo lo que queda. Cuando se quieren ejecutar muchas máquinas a la vez, esa capa intermedia desperdicia recursos.

**Proxmox VE** (*Virtual Environment*) se instala **directamente sobre el hardware**, como si fuera el propio sistema operativo del equipo. No hay "Windows + programa de virtualización": hay un servidor dedicado únicamente a crear y gestionar máquinas virtuales. Todo el hardware (RAM, CPU, disco) se reparte entre ellas.

> **Analogía:** VirtualBox es como alquilar habitaciones dentro de una vivienda en la que ya vive alguien. Proxmox es un edificio construido solo para alquilar habitaciones.

![Proxmox frente a otras soluciones de virtualización](https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/1200_628_Proxmox-vs.-Vmware%402x.png)

### Cómo se maneja

Se administra desde el navegador, entrando en `https://IP-DEL-EQUIPO:8006`. También hay línea de comandos en el propio equipo.

![Interfaz web de Proxmox](https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/proxmox-intro.png)

---

## Conceptos clave

### Node

El equipo físico donde está instalado Proxmox.

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/images.jpg" alt="Node" width="506">

### VM (máquina virtual)

Un ordenador completo simulado, con su CPU asignada, RAM, disco virtual y sistema operativo instalado desde cero. Permite instalar cualquier sistema (Linux, Windows, pfSense, Kali...).

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/virtual_machines.png" alt="Máquinas virtuales" width="548">

### LXC (contenedor)

Un contenedor Linux **comparte el kernel** del equipo anfitrión, por lo que arranca en segundos y consume mucho menos que una VM. A cambio, solo puede ejecutar sistemas Linux y el aislamiento es menor.

| | VM (KVM) | Contenedor (LXC) |
|---|---|---|
| Kernel | Propio | Compartido con el host |
| Sistemas posibles | Cualquiera | Solo Linux |
| Consumo | Mayor | Menor |
| Arranque | Más lento | Muy rápido |
| Aislamiento | Alto | Medio |

### Storage

Dónde se guardan los discos virtuales, las ISOs, las plantillas y las copias de seguridad.

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/shared_storage_XenServer_blog_virtualizacion.png" alt="Storage" width="346">

### Bridge

Un **bridge** es un **switch virtual**. Cuando se conecta la tarjeta de red de una VM a un bridge, esa VM queda en la misma red que cualquier otra VM conectada al mismo bridge.

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/vn-Bridged-Mode-Diagram.png" alt="Bridge" width="563">

### Snapshot

Una **foto congelada del estado** de una VM. Si algo se rompe probando, se restaura y se vuelve atrás en segundos sin reinstalar nada.

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/vmware-snapshot-monitoring-banner.svg" alt="Snapshot" width="531">

### Template

Una VM "maestra" ya configurada que se **clona** para crear VMs nuevas rápidamente. Ejemplo: instalar Ubuntu Server una vez, dejarlo limpio y actualizado, convertirlo en template y clonar desde ahí cada vez que haga falta una VM nueva.

<img src="https://github.com/alexnaranjooo/APUNTES-ASIX/raw/main/docs/.gitbook/assets/1069465476.svg" alt="Template" width="513">

---

## En técnico

### Tecnologías que hay debajo

- **KVM** (*Kernel-based Virtual Machine*): módulo del kernel de Linux que convierte el procesador en un hipervisor usando las extensiones de virtualización del hardware (Intel VT-x / AMD-V; hay que activarlas en la BIOS).
- **QEMU**: emula el resto del hardware de la VM (disco, red, tarjeta gráfica...).
- **LXC**: contenedores del kernel de Linux (namespaces y cgroups).

### Requisitos y puntos de partida

- CPU con virtualización por hardware activada en la BIOS.
- RAM suficiente: la suma de lo asignado a las VMs más lo que usa el propio Proxmox. Para laboratorios con muchas VMs se recomienda bastante memoria (16 GB o más).
- Disco rápido (SSD) para los discos virtuales.
- Proxmox instalado dentro de otra VM (virtualización anidada) funciona para pruebas, pero con peor rendimiento.

### Repositorios

Por defecto Proxmox trae activado el repositorio *enterprise*, que requiere suscripción. En laboratorios sin suscripción se desactiva y se usa el repositorio **no-subscription** para poder actualizar.

```bash
apt update && apt full-upgrade -y
pveversion          # versión instalada
```

### Almacenamiento

| Tipo | Descripción |
|---|---|
| `local` | Directorio del disco del sistema. Guarda ISOs, plantillas y backups. |
| `local-lvm` | LVM-thin. Guarda los discos de las VMs (permite snapshots). |
| ZFS | Sistema de ficheros avanzado con snapshots, compresión y redundancia. |
| NFS / SMB / iSCSI | Almacenamiento en red compartido. |

```bash
pvesm status        # estado de los storages
```

### Redes

La red se define en `/etc/network/interfaces`. Por defecto hay un bridge `vmbr0` enlazado a la tarjeta física:

```
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
```

Un bridge **sin puertos físicos** es una red interna aislada del exterior:

```
auto vmbr1
iface vmbr1 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

Para trabajar con VLANs, se hace el bridge *VLAN-aware*; después cada interfaz de VM puede llevar su etiqueta (campo *VLAN Tag* en la interfaz web):

```
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

Para aplicar los cambios sin reiniciar: `ifreload -a`.

> El *gateway* por defecto se define **una sola vez** en el nodo, aunque haya varios bridges.

### Comandos útiles (en el nodo)

| Comando | Para qué sirve |
|---|---|
| `qm list` | Lista las VMs. |
| `qm start 100` / `qm shutdown 100` / `qm stop 100` | Arrancar, apagar con ACPI, apagar forzado. |
| `qm snapshot 100 limpio` | Crea un snapshot llamado `limpio`. |
| `qm rollback 100 limpio` | Vuelve al snapshot. |
| `qm template 100` | Convierte la VM en template. |
| `qm clone 100 101 --name nueva --full` | Clona (copia completa) un template. |
| `pct list` | Lista los contenedores LXC. |
| `vzdump 100 --mode snapshot --compress zstd --storage local` | Copia de seguridad de la VM. |

### Buenas prácticas

- Instalar el **QEMU Guest Agent** en las VMs Linux (`apt install qemu-guest-agent`) y activarlo en las opciones de la VM: permite apagados limpios y ver la IP desde la interfaz.
- Usar controladores **VirtIO** para disco y red: dan mejor rendimiento que los emulados.
- Hacer un **snapshot antes de cada cambio importante**.
- Crear templates limpios y clonarlos, en lugar de instalar cada VM desde cero.
- Los snapshots **no son backups**: viven en el mismo disco. Para copias reales, ver [Backup](backup.md).

---

## Resumen

- Proxmox convierte un equipo en un servidor de virtualización dedicado (KVM para VMs, LXC para contenedores).
- Se gestiona desde el navegador (`:8006`) o por línea de comandos (`qm`, `pct`, `pvesm`, `vzdump`).
- Los **bridges** son switches virtuales; con *VLAN-aware* soportan VLANs.
- **Snapshots** para volver atrás y **templates** para desplegar rápido.

## En HoneyNet

Proxmox **no forma parte del servicio que vende HoneyNet**: es el **laboratorio** con el que simulamos la infraestructura del cliente. Un único equipo del aula aloja todas las máquinas virtuales (la red del cliente, la solución de HoneyNet y un atacante de pruebas). En un cliente real, su infraestructura ya existiría. Los snapshots permiten restaurar un honeypot comprometido y los bridges separan las zonas de red.

## Referencias

- Documentación oficial: <https://pve.proxmox.com/pve-docs/>
- Configuración de red: <https://pve.proxmox.com/wiki/Network_Configuration>
- Apuntes ASIX, Bloque 1: <https://github.com/alexnaranjooo/APUNTES-ASIX/blob/main/docs/honeynet/f0-profundizacion-completa/bloque-1-proxmox.md>

[← Teoría](README.md)
