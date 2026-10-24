# Seguridad de una aplicación web (PHP + MariaDB)

## En simple

### Por qué importa

Una aplicación web recibe datos de **cualquiera**: un formulario, una URL, una cookie. Si el programa se fía de esos datos, un atacante puede **colarle órdenes** (inyección), **robar sesiones** o **leer credenciales**. La seguridad web consiste en no confiar nunca en lo que llega de fuera y en limitar el daño si algo falla.

> **Analogía:** un restaurante con una ventanilla de pedidos. Si el cocinero cumple cualquier nota que le pasen sin leerla, alguien puede pedirle que abra la caja. Hay que **leer, validar y limitar** cada pedido.

### Las ideas clave

- **Nunca mezclar los datos del usuario con las órdenes**, como en las consultas SQL.
- **Escapar** lo que se muestra en pantalla.
- **No guardar contraseñas**, sino su huella (hash).
- **Mantener los secretos fuera del código.**
- **Mínimo privilegio** para la base de datos.

---

## Conceptos clave

| Término | Significado |
|---|---|
| **Inyección SQL (SQLi)** | El atacante escribe código SQL en un campo y la aplicación lo ejecuta. |
| **XSS** | El atacante consigue que su script se ejecute en el navegador de otros usuarios. |
| **Consulta preparada** | Consulta en la que la estructura y los datos van separados. |
| **PDO** | Biblioteca de PHP para acceder a bases de datos de forma segura y uniforme. |
| **Hash de contraseña** | Huella irreversible de una contraseña, hecha con un algoritmo lento (bcrypt, argon2id). |
| **Sesión** | Estado de un usuario identificado entre peticiones. |
| **CSRF** | Hacer que un usuario identificado ejecute una acción sin querer. |
| **`.env`** | Archivo con credenciales, fuera de la carpeta pública y del control de versiones. |
| **Mínimo privilegio** | Dar solo los permisos estrictamente necesarios. |

---

## En técnico

### Consultas preparadas con PDO (contra la inyección SQL)

**Mal** (concatenar datos del usuario):

```php
$sql = "SELECT * FROM usuarios WHERE nombre_usuario = '" . $_POST['usuario'] . "'";   // vulnerable
```

**Bien** (la consulta y los datos van por separado):

```php
$pdo = new PDO($dsn, $usuario, $clave, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES => false,
]);

$stmt = $pdo->prepare(
    'INSERT INTO mensajes_contacto (nombre, apellidos, telefono, mensaje)
     VALUES (:nombre, :apellidos, :telefono, :mensaje)'
);
$stmt->execute([
    ':nombre'    => $_POST['nombre'],
    ':apellidos' => $_POST['apellidos'],
    ':telefono'  => $_POST['telefono'],
    ':mensaje'   => $_POST['mensaje'],
]);
```

### Validar la entrada

Comprobar longitud y formato **antes** de guardar, de acuerdo con los límites de la base de datos:

```php
$nombre = trim($_POST['nombre'] ?? '');
if ($nombre === '' || mb_strlen($nombre) > 50) {
    $errores[] = 'El nombre es obligatorio y admite hasta 50 caracteres.';
}
```

### Escapar la salida (contra el XSS)

Todo lo que viene del usuario y se muestra en una página debe escaparse:

```php
echo htmlspecialchars($mensaje, ENT_QUOTES, 'UTF-8');
```

> Esto es especialmente importante con datos que escribe un atacante, como los usuarios y comandos que registra un honeypot: **no se muestran en bruto** en páginas propias, sino a través de herramientas que los escapan (como los paneles de Grafana).

### Contraseñas

Nunca se guarda la contraseña, sino su huella:

```php
// Al crear el usuario
$hash = password_hash($contrasena, PASSWORD_DEFAULT);       // bcrypt o argon2id según versión

// Al iniciar sesión
if (password_verify($contrasena, $hash_guardado)) {
    // correcto
}
```

El campo de la base de datos debe tener sitio de sobra (por ejemplo `VARCHAR(255)`).

### Sesiones y login

```php
session_start();
// Tras validar las credenciales:
session_regenerate_id(true);                    // evita la fijación de sesión
$_SESSION['usuario'] = $id_usuario;
```

- Mensajes de error **genéricos** ("usuario o contraseña incorrectos"), sin decir cuál falló.
- Cookies con las opciones `HttpOnly`, `Secure` y `SameSite`.
- Proteger los formularios sensibles con un **token CSRF**.
- Cerrar sesión destruyendo la sesión en el servidor.

### Credenciales fuera del código

```
# /var/www/sitio/.env  (fuera de public/, permisos 640, en .gitignore)
DB_HOST=192.168.20.11
DB_NAME=honeypot_web
DB_USER=web_app
DB_PASS=contraseña_larga_y_aleatoria
```

```bash
sudo chown root:www-data /var/www/sitio/.env
sudo chmod 640 /var/www/sitio/.env
echo ".env" >> .gitignore
```

El servidor web debe **bloquear el acceso a los archivos ocultos** (ver [nginx](nginx.md)).

### Usuario de base de datos con mínimos privilegios

La aplicación no se conecta como administrador, sino con un usuario que solo puede lo imprescindible y solo desde el servidor web:

```sql
CREATE USER 'web_app'@'192.168.60.10' IDENTIFIED BY 'contraseña_larga_y_aleatoria';
GRANT SELECT ON honeypot_web.planes TO 'web_app'@'192.168.60.10';
GRANT INSERT ON honeypot_web.mensajes_contacto TO 'web_app'@'192.168.60.10';
FLUSH PRIVILEGES;
```

### Otras medidas

- **Límite de peticiones** al formulario (ver [nginx](nginx.md)).
- **Cabeceras de seguridad** (`X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`).
- **Actualizar** PHP, nginx y las bibliotecas.
- **No mostrar errores detallados** al usuario; registrarlos en el servidor.
- Probar la aplicación contra ataques conocidos en un entorno aislado (por ejemplo, con **DVWA** como práctica de los fallos más habituales).

---

## Resumen

- No confiar en nada que llegue del exterior: **validar, preparar y escapar**.
- Consultas preparadas contra SQLi; `htmlspecialchars()` contra XSS; `password_hash()` para las contraseñas.
- Credenciales en `.env` fuera de la carpeta pública y fuera de Git.
- Usuario de base de datos con **mínimo privilegio**.

## En HoneyNet

La web de la empresa (nginx + PHP 8.4 con PHP-FPM y MariaDB) aplica todas estas medidas: consultas preparadas con PDO, credenciales en un `.env` fuera de `public/`, un usuario `web_app` con permisos mínimos y solo desde `ct-web`, login de administrador y paneles de Grafana en lugar de logs en bruto. Ver [Seguridad](../Introducci%C3%B3n/arquitectura-del-software/6.-seguridad.md).

## Referencias

- OWASP Top 10: <https://owasp.org/www-project-top-ten/>
- Manual de PHP, PDO: <https://www.php.net/manual/es/book.pdo.php>
- DVWA: <https://github.com/digininja/DVWA>

[← Teoría](README.md)
