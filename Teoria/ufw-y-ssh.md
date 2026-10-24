# UFW y SSH con claves

## En simple

### Qué es UFW

**UFW** (*Uncomplicated Firewall*) es una herramienta que permite configurar el firewall de un equipo Linux con comandos sencillos. Por debajo usa el firewall del kernel (netfilter), pero evita escribir las reglas complejas a mano.

> **Analogía:** el portero de un edificio con una lista de visitas autorizadas. Por defecto, no entra nadie; solo pasan las personas que están en la lista, y a las plantas que se les permite.

Se usa en **cada servidor** para aplicar el **mínimo privilegio**: el equipo solo acepta el tráfico que necesita, y solo de quien debe. Así, aunque otra defensa falle, cada servidor sigue protegido.

### Qué es SSH y por qué usar claves

**SSH** es el protocolo para administrar un equipo de forma remota y cifrada. Entrar con **contraseña** es el punto débil: se puede adivinar o probar masivamente (fuerza bruta). Con **claves** se usa un par de ficheros matemáticamente relacionados:

- La **clave privada**: se queda en el equipo del administrador y no se comparte nunca.
- La **clave pública**: se copia al servidor, que la usa para comprobar la identidad.

> **Analogía:** la contraseña es una frase que cualquiera puede oír; la clave es una llave física única que solo tú tienes.

---

## Conceptos clave

| Término | Significado |
|---|---|
| **Denegar por defecto** | Bloquear todo lo que no esté permitido explícitamente. |
| **Mínimo privilegio** | Dar solo los accesos estrictamente necesarios. |
| **Regla con origen** | Permitir un puerto solo desde una IP o red concreta. |
| **Par de claves** | Clave privada + clave pública. |
| **ed25519** | Algoritmo moderno y seguro para claves SSH. |
| **Fuerza bruta** | Probar muchas contraseñas hasta acertar. |
| **Defensa en profundidad** | Varias capas: un firewall de red (pfSense) y otro en cada servidor (UFW). |

---

## En técnico

### UFW: instalación y política por defecto

```bash
sudo apt install ufw
sudo ufw default deny incoming        # bloquear toda entrada por defecto
sudo ufw default allow outgoing       # permitir la salida
```

### Permitir solo lo necesario

```bash
# Un puerto abierto a cualquiera (por ejemplo, la web)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Un puerto solo desde una red (por ejemplo, SSH desde la zona de gestión)
sudo ufw allow from 192.168.30.0/24 to any port 22 proto tcp

# Un puerto solo desde una IP concreta (por ejemplo, base de datos solo para el servidor web)
sudo ufw allow from 192.168.60.10 to any port 3306 proto tcp
```

### Activar y comprobar

```bash
sudo ufw enable
sudo ufw status verbose
sudo ufw status numbered              # lista con números, útil para borrar
sudo ufw delete 3                     # borrar la regla número 3
sudo ufw logging on                   # registrar los bloqueos
```

> **Cuidado al trabajar por SSH:** antes de `ufw enable`, permite siempre el puerto 22 desde tu red; si no, te quedas sin acceso. Usa también una segunda sesión abierta mientras pruebas.

### Ejemplo por tipo de servidor

| Servidor | Entrada permitida | Origen |
|---|---|---|
| Web | 80 y 443/tcp | Cualquiera |
| Base de datos | 3306/tcp | Solo el servidor web |
| DNS | 53/tcp y 53/udp | Las redes que lo usan |
| Todos | 22/tcp | Solo la red de gestión |

Además del firewall, conviene **limitar en el propio servicio** quién puede conectar. Por ejemplo, que la base de datos escuche solo en su IP (`bind-address`), de modo que el firewall no sea la única barrera.

### SSH con claves

```bash
# 1. En el equipo del administrador: generar el par de claves
ssh-keygen -t ed25519 -C "admin"

# 2. Copiar la clave pública al servidor
ssh-copy-id usuario@servidor

# 3. Comprobar que se entra sin contraseña
ssh usuario@servidor
```

Después, en el servidor, se endurece `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

```bash
sudo sshd -t                      # comprobar la sintaxis
sudo systemctl reload ssh         # aplicar (el servicio puede llamarse ssh o sshd)
```

> **Antes de cerrar la sesión actual**, abre una segunda terminal y comprueba que entras con la clave. Si te equivocas, evita quedarte fuera.

### Buenas prácticas

- Proteger la **clave privada** con una frase de paso (*passphrase*).
- Una clave por persona; **retirar** la clave pública de quien deje el equipo.
- No reutilizar claves entre entornos sensibles.
- Combinar con restricciones de red: SSH solo desde la zona de gestión.

---

## Resumen

- UFW aplica **denegar por defecto** y abre solo lo necesario, con origen concreto cuando es posible.
- SSH con claves elimina el riesgo de adivinar contraseñas; se desactivan la contraseña y el acceso directo de root.
- Cada servidor tiene su firewall además del firewall de red: **defensa en profundidad**.

## En HoneyNet

Cada contenedor del laboratorio (`ct-web`, `ct-db`, `ct-dns-dhcp` y `ct-backup`) usa UFW con denegar por defecto; por ejemplo, la base de datos solo acepta conexiones del servidor web. El SSH de todos los servidores solo se acepta desde la VLAN de gestión y solo con claves. Ver [Seguridad](../Introducci%C3%B3n/arquitectura-del-software/6.-seguridad.md).

## Referencias

- UFW: <https://help.ubuntu.com/community/UFW>
- `man ufw`, `man sshd_config`

[← Teoría](README.md)
