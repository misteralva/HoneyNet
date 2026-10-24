# Backup

## En simple

### Qué es y por qué importa

Una **copia de seguridad** (*backup*) es una copia de los datos guardada en otro lugar, para poder recuperarlos si los originales se pierden, se corrompen o se borran por error. Las causas más comunes son fallos de disco, errores humanos, malware y ransomware.

> **Analogía:** una copia de seguridad es la llave de repuesto de tu casa. Solo la valoras el día que pierdes la original, y de nada sirve si la guardas dentro de la propia casa.

> Una copia de seguridad que nunca se ha probado a restaurar **no es una copia de seguridad**, es una esperanza.

### Los dos números que definen una estrategia

- **RPO** (*Recovery Point Objective*): cuántos datos se pueden permitir perder, medido en tiempo. Un RPO de 24 horas significa que vale con una copia diaria.
- **RTO** (*Recovery Time Objective*): cuánto tiempo se puede tardar en volver a funcionar tras un fallo.

### Tipos de copia

| Tipo | Qué copia | Ventaja | Inconveniente |
|---|---|---|---|
| **Completa** | Todo, cada vez | Restaurar es sencillo | Ocupa mucho espacio y tiempo |
| **Incremental** | Solo lo que cambió desde la **última copia** (de cualquier tipo) | Rápida y ligera | Para restaurar hacen falta la completa y todas las incrementales |
| **Diferencial** | Lo que cambió desde la **última completa** | Restaurar necesita solo dos piezas | Crece cada día hasta la próxima completa |

```mermaid
flowchart LR
    C["Completa (domingo)"] --> I1["Incremental (lunes)"] --> I2["Incremental (martes)"] --> I3["Incremental (miércoles)"]
```

### La regla 3-2-1

- **3** copias de los datos (el original y dos copias).
- **2** tipos de soporte distintos (por ejemplo, disco local y almacenamiento en red).
- **1** copia **fuera del sitio** (otra ubicación o la nube).

### Otras ideas importantes

- **Retención**: cuánto tiempo se guardan las copias antiguas (por ejemplo, 7 diarias, 4 semanales).
- **Automatización**: las copias manuales se olvidan; hay que programarlas.
- **Verificación y restauración de prueba**: comprobar periódicamente que se puede recuperar.
- **Cifrado**: si la copia sale del sitio, debe ir cifrada.
- **Snapshot ≠ backup**: un snapshot vive en el mismo almacenamiento que el original; si falla el disco, se pierden ambos.

---

## En técnico

### Herramienta 1: `tar` (empaquetar y comprimir)

`tar` junta muchos ficheros en uno solo; con `gzip` además los comprime (`.tar.gz`).

```bash
# Crear una copia comprimida
tar -czpf copia_$(date +%F).tar.gz /ruta/a/copiar

# Ver el contenido sin extraer
tar -tzvf copia_2026-10-14.tar.gz

# Restaurar en una carpeta concreta
tar -xzpf copia_2026-10-14.tar.gz -C /ruta/destino

# Excluir ficheros
tar -czpf copia.tar.gz --exclude='*.tmp' --exclude='cache/' /ruta/a/copiar
```

| Opción | Significado |
|---|---|
| `-c` | Crear un archivo. |
| `-x` | Extraer. |
| `-t` | Listar el contenido. |
| `-z` | Comprimir/descomprimir con gzip. |
| `-v` | Mostrar lo que va haciendo. |
| `-p` | Conservar permisos. |
| `-f` | Nombre del archivo (siempre va seguido del nombre). |
| `-C` | Cambiar al directorio indicado antes de operar. |

Comprobar la integridad con un hash:

```bash
sha256sum copia_2026-10-14.tar.gz > copia_2026-10-14.tar.gz.sha256
sha256sum -c copia_2026-10-14.tar.gz.sha256
```

### Herramienta 2: `rsync` (sincronizar)

`rsync` copia solo **las diferencias** entre origen y destino, por lo que las copias repetidas son muy rápidas. Funciona en local y por red a través de SSH.

```bash
# Copia local
rsync -avh /carpeta_origen/ /carpeta_destino/

# Simulación (no copia nada, solo muestra lo que haría)
rsync -avhn --delete /carpeta_origen/ /carpeta_destino/
```

| Opción | Significado |
|---|---|
| `-a` | Modo archivo: recursivo y conserva permisos, fechas, propietario y enlaces. |
| `-v` | Detallado. |
| `-h` | Tamaños legibles. |
| `-z` | Comprime durante la transferencia (útil por red). |
| `-n` | Simulación (*dry run*). |
| `--delete` | Borra en el destino lo que ya no existe en el origen (espejo exacto). |
| `--exclude` | Excluye patrones. |
| `-P` | Muestra progreso y permite reanudar. |
| `-e ssh` | Usa SSH como transporte. |

> **La barra final importa.** `rsync -a /origen/ /destino/` copia el **contenido** de `origen` dentro de `destino`. `rsync -a /origen /destino/` crea `/destino/origen`.

> Usar `--delete` sin haber probado antes con `-n` puede borrar datos del destino. Hacer siempre la simulación primero.

### Copia entre dos máquinas (MV1 → MV2)

```
MV1 (origen)  ──────────────►  MV2 (destino)
user1@ip_MV1:/carpeta_origen/   user2@ip_MV2:/carpeta_destino/
```

`rsync` **no permite que origen y destino sean ambos remotos** en el mismo comando. Se ejecuta desde una de las dos máquinas, de una de estas formas:

**Desde MV1, empujando (*push*) hacia MV2**

```bash
rsync -avz -e ssh /carpeta_origen/ user2@ip_MV2:/carpeta_destino/
```

**Desde MV2, trayendo (*pull*) desde MV1**

```bash
rsync -avz -e ssh user1@ip_MV1:/carpeta_origen/ /carpeta_destino/
```

Para que sea automático hace falta que SSH no pida contraseña, usando **claves**:

```bash
# En la máquina que lanza el comando
ssh-keygen -t ed25519
ssh-copy-id user2@ip_MV2        # copia la clave pública al otro equipo
ssh user2@ip_MV2                # debe entrar sin pedir contraseña
```

Si SSH usa otro puerto: `rsync -avz -e "ssh -p 2222" ...`.

### Copias incrementales con enlaces duros

`--link-dest` permite guardar varias fechas como carpetas completas, pero los ficheros que no han cambiado ocupan espacio una sola vez:

```bash
rsync -a --delete --link-dest=/backups/ayer /datos/ /backups/hoy/
```

### Un script de backup completo

```bash
#!/bin/bash
# /usr/local/bin/backup.sh
set -euo pipefail

ORIGEN="/datos"
DESTINO="/backups"
FECHA=$(date +%F)
ARCHIVO="$DESTINO/datos_$FECHA.tar.gz"
RETENCION_DIAS=7

mkdir -p "$DESTINO"

# 1. Crear la copia
tar -czpf "$ARCHIVO" "$ORIGEN"

# 2. Guardar su huella para poder verificarla
sha256sum "$ARCHIVO" > "$ARCHIVO.sha256"

# 3. Enviarla a otra máquina
rsync -az -e ssh "$ARCHIVO" "$ARCHIVO.sha256" user2@ip_MV2:/carpeta_destino/

# 4. Borrar copias locales con más de N días
find "$DESTINO" -name 'datos_*.tar.gz*' -mtime +"$RETENCION_DIAS" -delete

echo "$(date '+%F %T') backup correcto" >> /var/log/backup.log
```

```bash
sudo chmod +x /usr/local/bin/backup.sh
```

### Programarlo: `cron` y `systemd`

Hay dos formas de ejecutar tareas de manera automática en Linux, y cumplen la misma función:

```
CRONTAB  ⇄  SYSTEMCTL (systemd timers)
```

#### Opción A: `crontab`

```bash
crontab -e        # editar las tareas del usuario
crontab -l        # listar
```

Formato de una línea: cinco campos de tiempo y el comando.

```
┌───────── minuto (0-59)
│ ┌─────── hora (0-23)
│ │ ┌───── día del mes (1-31)
│ │ │ ┌─── mes (1-12)
│ │ │ │ ┌─ día de la semana (0-7, 0 y 7 = domingo)
│ │ │ │ │
* * * * *  comando
```

Ejemplos:

```
0 2 * * *      /usr/local/bin/backup.sh              # todos los días a las 02:00
30 3 * * 0     /usr/local/bin/backup.sh              # domingos a las 03:30
*/15 * * * *   /usr/local/bin/script.sh              # cada 15 minutos
@reboot        /usr/local/bin/script.sh              # al arrancar
0 2 * * *      /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

> Cron ejecuta con un entorno muy reducido: usar **rutas absolutas** en comandos y scripts.

#### Opción B: `systemd` timers (con `systemctl`)

Se necesitan dos ficheros: un **service** (qué ejecutar) y un **timer** (cuándo).

`/etc/systemd/system/backup.service`

```ini
[Unit]
Description=Copia de seguridad

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

`/etc/systemd/system/backup.timer`

```ini
[Unit]
Description=Ejecuta la copia de seguridad cada día a las 02:00

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers                 # ver los timers y su próxima ejecución
sudo systemctl start backup.service   # lanzarlo a mano para probarlo
journalctl -u backup.service          # ver los logs
systemd-analyze calendar "*-*-* 02:00:00"   # comprobar una expresión de calendario
```

`Persistent=true` hace que, si el equipo estaba apagado a la hora programada, la tarea se ejecute al encenderse.

#### Comparativa

| | `cron` | `systemd` timer |
|---|---|---|
| Configuración | Una línea en `crontab` | Dos ficheros (`.service` y `.timer`) |
| Logs | Hay que redirigirlos a mano | Automáticos con `journalctl` |
| Si el equipo estaba apagado | La tarea se pierde (salvo `anacron`) | `Persistent=true` la recupera |
| Dependencias y orden | No | Sí (`After=`, `Requires=`...) |
| Sencillez | Muy sencillo | Más verboso, más potente |

### Probar la restauración

```bash
mkdir /tmp/prueba && tar -xzpf /backups/datos_2026-10-14.tar.gz -C /tmp/prueba
diff -r /datos /tmp/prueba/datos && echo "La copia coincide"
```

### Backups en Proxmox

Proxmox incluye su propia herramienta, `vzdump`, para copiar VMs y contenedores completos, programable desde la interfaz web (*Datacenter → Backup*):

```bash
vzdump 100 --mode snapshot --compress zstd --storage local
```

| Modo | Comportamiento |
|---|---|
| `stop` | Apaga la VM durante la copia. El más consistente. |
| `suspend` | Pausa brevemente la VM. |
| `snapshot` | Copia con la VM encendida. Cómodo y con casi ningún tiempo de parada. |

Existe también **Proxmox Backup Server**, un servidor específico con deduplicación y copias incrementales.

---

## Resumen

- Una copia sirve si es **automática, verificada y está en otro sitio** (regla 3-2-1).
- `tar` empaqueta y comprime; `rsync` sincroniza solo las diferencias, también por SSH.
- `cron` y los `systemd timers` cumplen la misma función de programar tareas; los timers aportan logs y recuperación tras apagados.
- Entre dos máquinas, `rsync` se lanza desde una de ellas en modo *push* o *pull*, con claves SSH.

## En HoneyNet

Hay **dos niveles de copias** distintos:

- **Copias del servicio:** el contenedor `ct-backup` (VLAN 20) hace copias nocturnas (a las 03:00) de la base de datos y de las configuraciones con `rsync` y `cron`. El acceso a los servidores es solo por SSH con claves desde la VLAN de gestión.
- **Copias del proyecto (nosotros, como estudiantes):** como todo vive en dos PCs prestados, si falla un disco se pierde trabajo. Por eso se hacen copias programadas de las máquinas virtuales en Proxmox, se **exporta la configuración de pfSense y del switch** y la documentación y el código están en **GitHub**. Es la regla 3-2-1 aplicada a nuestro trabajo.

## Referencias

- `man tar`, `man rsync`, `man 5 crontab`, `man systemd.timer`
- Proxmox Backup y Restore: <https://pve.proxmox.com/wiki/Backup_and_Restore>

[← Teoría](README.md)
