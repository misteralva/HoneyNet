# Servidor web (nginx)

## En simple

### Qué es un servidor web

Un **servidor web** es el programa que entrega al navegador los archivos de una página web (HTML, CSS, JavaScript e imágenes) cuando alguien la solicita. **nginx** (se pronuncia "engine-x") es uno de los servidores web más usados: es ligero, rápido y estable.

> **Analogía:** un camarero. El navegador hace un pedido (una dirección web) y nginx busca en la cocina (la carpeta con los archivos) y lo lleva a la mesa.

Además de servir páginas, nginx puede **redirigir**, cifrar con HTTPS, limitar peticiones y actuar como **proxy inverso** delante de otras aplicaciones.

### Por qué importa en seguridad

Un servidor web **está expuesto por diseño**: recibe peticiones de cualquiera. Por eso hay que:

- Colocarlo en una **zona de red propia**, separada del resto.
- **Endurecerlo** (ocultar información, permitir solo lo necesario, limitar abusos).
- **Vigilar sus registros** para detectar comportamientos extraños.

---

## Conceptos clave

| Término | Significado |
|---|---|
| **HTTP / HTTPS** | Protocolo web / su versión cifrada (puertos 80 y 443). |
| **Certificado TLS** | Documento que permite el cifrado HTTPS y acredita la identidad del servidor. |
| **Certificado autofirmado** | Certificado generado por uno mismo: cifra, pero los navegadores no lo reconocen como de confianza. |
| **CA interna** | Autoridad de certificación propia de una organización, en la que sus equipos confían. |
| **Server block** | Bloque de configuración de nginx que define un sitio web. |
| **Cabeceras de seguridad** | Instrucciones que el servidor envía al navegador para reducir riesgos. |
| **Proxy inverso** | Servidor que recibe las peticiones y las reenvía a otro servidor. |
| **Rate limiting** | Limitar cuántas peticiones puede hacer un cliente en un tiempo dado. |
| **PHP-FPM** | Proceso que ejecuta código PHP para que nginx pueda servir páginas dinámicas. |

---

## En técnico

### Instalación

```bash
sudo apt install nginx
sudo systemctl enable --now nginx
sudo nginx -t                    # comprobar la configuración
sudo systemctl reload nginx      # aplicar cambios sin cortar conexiones
```

Estructura habitual en Debian: la configuración general está en `/etc/nginx/nginx.conf`, los sitios disponibles en `/etc/nginx/sites-available/` (se activan con un enlace simbólico en `sites-enabled/`), y los registros en `/var/log/nginx/`.

### Un sitio sencillo con HTTPS

`/etc/nginx/sites-available/honeypot`

```nginx
server {
    listen 80;
    server_name www.honeypot.lan;
    return 301 https://$host$request_uri;        # redirigir a HTTPS
}

server {
    listen 443 ssl;
    server_name www.honeypot.lan;

    ssl_certificate     /etc/ssl/certs/honeypot.crt;
    ssl_certificate_key /etc/ssl/private/honeypot.key;

    root  /var/www/honeypot.lan/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/honeypot /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### Certificado para un nombre interno (`.lan`, `.local`)

**Let's Encrypt no emite certificados para nombres internos como `honeypot.lan`**, porque no son dominios públicos. Hay dos opciones:

- **Certificado autofirmado** (rápido, para laboratorio):

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/honeypot.key \
  -out /etc/ssl/certs/honeypot.crt \
  -subj "/CN=www.honeypot.lan"
```

- **Una CA interna**, en la que se hace confiar a los equipos de la organización (más realista en una empresa).

### Endurecimiento (*hardening*)

```nginx
# En el bloque http { } de nginx.conf
server_tokens off;                       # no mostrar la versión de nginx
limit_req_zone $binary_remote_addr zone=contacto:10m rate=5r/m;

# Dentro del server { }
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

location /contacto {
    limit_except GET POST { deny all; }  # solo los métodos necesarios
    limit_req zone=contacto burst=3;     # limitar el formulario
}
```

Otras medidas habituales:

- **Permitir solo los métodos HTTP necesarios.**
- **Limitar el tamaño** de las peticiones (`client_max_body_size`).
- **Rotar los logs** (en Debian, `logrotate` ya viene configurado para nginx).
- Mantener nginx **actualizado**.
- No dejar archivos de copia, de prueba o ficheros ocultos en la carpeta pública.

### Formulario de contacto

Hay dos enfoques:

- **Sin servidor detrás:** la opción más simple; el formulario no procesa datos (por ejemplo, abre el correo del usuario o muestra solo información de contacto).
- **Con PHP y base de datos:** se usa **PHP-FPM** y se guardan las solicitudes en una base de datos **local del propio servidor web**, para no abrir otro flujo de red hacia otros servidores.

### PHP con PHP-FPM

nginx no ejecuta PHP: pasa los archivos `.php` a **PHP-FPM**, un proceso aparte, a través de un socket Unix.

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.4-fpm.sock;
}

# Impedir el acceso a archivos ocultos (.env, .git...)
location ~ /\. {
    deny all;
}
```

El código y las credenciales (`.env`) deben quedar **fuera de la carpeta pública** que sirve nginx.

### Proxy inverso y restricción por IP

Un **proxy inverso** recibe la petición y la reenvía a otro servidor. Sirve, por ejemplo, para mostrar un panel de otra aplicación (como Grafana) dentro de la misma web. Combinado con `allow` y `deny`, se limita quién puede usarlo:

```nginx
location /grafana/ {
    allow 192.168.30.0/24;    # zona de gestión
    allow 192.168.40.0/24;    # usuarios
    deny all;                 # todo lo demás
    proxy_pass http://192.168.30.10:3000/;
}
```

La aplicación de destino debe estar preparada para servirse bajo esa ruta (en Grafana, con `serve_from_sub_path`). Esta restricción es una barrera de red **adicional**: la zona de administración debe tener también su propio login.

### Vigilar los registros

nginx registra cada petición en `/var/log/nginx/access.log` y los errores en `error.log`. Un **agente de Wazuh** en el servidor puede leer esos ficheros y generar alertas ante patrones sospechosos (muchos errores 404, intentos de inyección, peticiones a rutas raras...). Ver [Monitorización y respuesta](wazuh-grafana.md).

---

## Resumen

- nginx entrega páginas web y puede cifrar, redirigir y limitar peticiones.
- Un servidor web debe vivir en su **propia zona de red** y estar **endurecido**.
- Para nombres internos (`.lan`, `.local`) se usa un certificado **autofirmado** o de una **CA interna**.
- Sus registros se vigilan con un agente del SIEM.

## En HoneyNet

`ct-web` es un contenedor LXC de la **VLAN 60** con nginx + PHP-FPM que sirve la **web de la empresa** (dominio `honeypot.lan`). Pasa los `.php` a PHP-FPM, guarda los mensajes de contacto en MariaDB (`ct-db`) y hace de **proxy hacia Grafana** en `/grafana/`, limitado por IP a la red interna. Solo puede enviar logs a Wazuh y consultar su base de datos y el DNS.

## Referencias

- Documentación de nginx: <https://nginx.org/en/docs/>
- Mozilla Web Security Guidelines: <https://infosec.mozilla.org/guidelines/web_security>

[← Teoría](README.md)
