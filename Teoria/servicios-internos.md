# Servicios internos: DNS, DHCP y MySQL

Servicios que casi cualquier red de empresa necesita y que dan realismo a una infraestructura.

---

## DNS

### En simple

El **DNS** (*Domain Name System*) traduce **nombres** que las personas recuerdan (`www.ejemplo.com`) en **direcciones IP** que usan las máquinas (`203.0.113.10`).

> **Analogía:** la agenda de contactos del móvil. Tú buscas "Mamá" y el teléfono marca el número.

Dos papeles distintos:

- **Servidor autoritativo**: tiene los datos oficiales de una zona (por ejemplo, el dominio de la empresa) y responde sobre ella.
- **Servidor recursivo (resolvedor)**: recibe consultas de los clientes y, si no sabe la respuesta, la busca preguntando a otros servidores.

### Conceptos clave

| Término | Significado |
|---|---|
| **Zona** | Parte del espacio de nombres que un servidor administra. |
| **Registro A / AAAA** | Nombre → IPv4 / IPv6. |
| **CNAME** | Alias de otro nombre. |
| **MX** | Servidor de correo del dominio. |
| **NS** | Servidores de nombres de la zona. |
| **PTR** | Resolución inversa: IP → nombre. |
| **SOA** | Datos de la zona (servidor principal, número de serie, tiempos). |
| **TTL** | Tiempo que una respuesta puede guardarse en caché. |
| **Maestro / esclavo** | Un servidor guarda la zona original y otro la replica automáticamente. |

### En técnico: Bind9 en Debian/Ubuntu

```bash
sudo apt install bind9 bind9-utils dnsutils
```

Ficheros principales (en `/etc/bind/`): `named.conf.options` (opciones generales) y `named.conf.local` (zonas propias).

**Declarar una zona** en `/etc/bind/named.conf.local`:

```
zone "empresa.local" {
    type master;
    file "/etc/bind/db.empresa.local";
};
```

**Fichero de zona** `/etc/bind/db.empresa.local`:

```
$TTL    604800
@       IN      SOA     ns1.empresa.local. admin.empresa.local. (
                        2026101401   ; Serial (cambiar en cada modificación)
                        604800       ; Refresh
                        86400        ; Retry
                        2419200      ; Expire
                        604800 )     ; TTL negativo

@       IN      NS      ns1.empresa.local.
ns1     IN      A       10.0.20.10
www     IN      A       10.0.20.20
bd      IN      A       10.0.20.30
correo  IN      CNAME   www
```

> Los nombres terminados en punto son absolutos. Olvidar el punto final es el error más habitual.

**Comprobar y aplicar:**

```bash
sudo named-checkconf
sudo named-checkzone empresa.local /etc/bind/db.empresa.local
sudo systemctl restart named        # o bind9
dig @10.0.20.10 www.empresa.local
```

Para permitir **reenvío** de consultas desconocidas a otro DNS, en `named.conf.options`: `forwarders { 1.1.1.1; };`.

---

## DHCP

### En simple

El **DHCP** (*Dynamic Host Configuration Protocol*) asigna **automáticamente** la configuración de red a los equipos que se conectan: dirección IP, máscara, puerta de enlace y servidores DNS. Evita configurar cada equipo a mano.

> **Analogía:** el mostrador de un hotel que entrega a cada huésped una llave de habitación (una IP) por el tiempo que se aloje.

### Conceptos clave

- **DORA**: las cuatro fases del proceso: *Discover* (el cliente busca servidores), *Offer* (el servidor ofrece una IP), *Request* (el cliente la pide) y *Acknowledge* (el servidor confirma).
- **Ámbito (*scope*)**: rango de IPs que el servidor puede repartir.
- **Concesión (*lease*)**: tiempo durante el que un cliente puede usar la IP.
- **Reserva**: una IP fija para una dirección MAC concreta.
- **Relay DHCP**: reenvía las peticiones a un servidor DHCP situado en otra red.

### En técnico: isc-dhcp-server

```bash
sudo apt install isc-dhcp-server
```

Indicar en qué interfaz escucha, en `/etc/default/isc-dhcp-server`:

```
INTERFACESv4="ens19"
```

Configuración en `/etc/dhcp/dhcpd.conf`:

```
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 10.0.40.0 netmask 255.255.255.0 {
    range 10.0.40.100 10.0.40.200;
    option routers 10.0.40.1;
    option domain-name-servers 10.0.20.10;
    option domain-name "empresa.local";
}

# Reserva por MAC
host impresora {
    hardware ethernet aa:bb:cc:dd:ee:ff;
    fixed-address 10.0.40.50;
}
```

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf      # comprobar la sintaxis
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
cat /var/lib/dhcp/dhcpd.leases              # concesiones activas
```

> El servidor tiene que tener una IP **estática** en la red que reparte. Si los clientes están en otra red, hace falta un relay DHCP.

> ISC ha anunciado el fin del mantenimiento de ISC DHCP y recomienda su sucesor, **Kea**. Para aprendizaje sigue siendo muy válido, pero para despliegues nuevos conviene valorar Kea.

---

## MySQL / MariaDB

### En simple

**MySQL** (y **MariaDB**, su bifurcación compatible, que es la que suelen instalar las distribuciones Debian/Ubuntu) es un **sistema gestor de bases de datos relacionales**: guarda la información ordenada en **tablas** y permite consultarla con el lenguaje **SQL**.

> **Analogía:** un archivador con carpetas (tablas) donde cada ficha (fila) tiene los mismos apartados (columnas), y un bibliotecario (SQL) que localiza lo que pidas.

### Conceptos clave

- **Base de datos**: conjunto de tablas relacionadas.
- **Tabla, fila, columna**: estructura de los datos.
- **Clave primaria**: columna que identifica de forma única cada fila.
- **Usuario y privilegios**: quién puede hacer qué sobre qué base de datos.
- **SQL**: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`...

### En técnico

**Instalación y seguridad inicial:**

```bash
sudo apt install mariadb-server
sudo mysql_secure_installation
sudo systemctl status mariadb
```

**Crear una base de datos, un usuario y una tabla de ejemplo:**

```sql
CREATE DATABASE registros CHARACTER SET utf8mb4;

CREATE USER 'app'@'10.0.20.%' IDENTIFIED BY 'una-clave-segura';
GRANT SELECT, INSERT ON registros.* TO 'app'@'10.0.20.%';
FLUSH PRIVILEGES;

USE registros;
CREATE TABLE eventos (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    fecha       DATETIME NOT NULL,
    ip_origen   VARCHAR(45) NOT NULL,
    usuario     VARCHAR(100),
    detalle     TEXT
);

INSERT INTO eventos (fecha, ip_origen, usuario, detalle)
VALUES (NOW(), '203.0.113.5', 'root', 'Intento de acceso fallido');

SELECT ip_origen, COUNT(*) AS intentos
FROM eventos
GROUP BY ip_origen
ORDER BY intentos DESC
LIMIT 10;
```

**Acceso remoto.** Por defecto el servidor solo escucha en local. Para permitir conexiones desde otras máquinas se cambia `bind-address` en el fichero de configuración (en MariaDB, por ejemplo `/etc/mysql/mariadb.conf.d/50-server.cnf`) y se reinicia el servicio. Hay que **restringir con el firewall** quién puede llegar al puerto 3306.

**Copia de seguridad de una base de datos:**

```bash
mysqldump -u root -p registros > registros_$(date +%F).sql
mysql -u root -p registros < registros_2026-10-14.sql     # restaurar
```

Ver [Backup](backup.md) para automatizarlo.

### Buenas prácticas

- Un **usuario por aplicación**, con los privilegios mínimos.
- No usar `root` desde las aplicaciones.
- Limitar con el firewall el acceso al puerto 3306.
- Hacer copias de seguridad periódicas y probar la restauración.

---

## Resumen

- **DNS** traduce nombres en IPs; con Bind9 se definen zonas con registros A, CNAME, NS, etc.
- **DHCP** reparte la configuración de red automáticamente (proceso DORA) y permite reservas.
- **MySQL/MariaDB** almacena datos en tablas; se administra con SQL, usuarios con privilegios mínimos y copias con `mysqldump`.

## En HoneyNet

DNS y DHCP forman parte de la red de la empresa cliente: son la infraestructura habitual de cualquier empresa. La base de datos MySQL/MariaDB, en cambio, es **propia de HoneyNet**: guarda los datos de los ataques capturados para los informes y los paneles.

## Referencias

- Bind9: <https://bind9.readthedocs.io/>
- ISC DHCP / Kea: <https://www.isc.org/>
- MariaDB: <https://mariadb.com/kb/en/documentation/>

[← Teoría](README.md)
