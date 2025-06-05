# Guía Setup Sitios Web Estáticos (Plantilla Genérica con www y Nginx en Easypanel)

Por: Guillermo Gutiérrez [https://www.linkedin.com/in/ggd79/](https://www.linkedin.com/in/ggd79/)

## INSTRUCCIONES

### 1. CREAR BOX EN EASYPANEL Y CONFIGURACIÓN INICIAL DE MÓDULOS

a. **Nombre de servicio:** Ej: `static-website`

b. **Preset:** no preset

c. Pulsar **"Install Repository"**.

d. Una vez instalado, **"Stop"** al contenedor/servicio.

e. **Módulos (Modules):** Añadir y configurar "Nginx" y "Domains".
    i. **Módulo NGINX (Configuración General del Servicio):**
        - Root document: `/code`
        - Habilitar y Guardar.
    ii. **Módulo DOMAINS (Para cada sitio web):**
        - Hostname: `tusitioweb1.com` (ej: `mi-primer-sitio.com`)
        - Path: `/tusitioweb1/`
          *(Este es el subdirectorio DENTRO de `/code` donde residirán los archivos de este sitio. Ej: Si tu hostname es `tusitioweb1.com`, el path podría ser `/tusitioweb1/`).*
        - Añadir y repetir para cada dominio que vayas a alojar (`tusitioweb2.com` con path `/tusitioweb2/`, etc.).

### 2. CONFIGURACIÓN DE NGINX (EDITAR Y PEGAR LA PLANTILLA)

En el módulo de Nginx de tu servicio en Easypanel, haz clic en **"Edit Config"**.
Borra cualquier configuración existente y pega la siguiente estructura.
Deberás reemplazar `tusitioweb1.com`, `www.tusitioweb1.com`, y la ruta `/code/tusitioweb1` (y así sucesivamente para `tusitioweb2`, etc.) con tus propios datos.

```nginx
# --- Servidor por defecto (Fallback) ---
server {
    listen 80 default_server;
    server_name _;
    root /code;

    index index.html index.htm; # Para sitios estáticos por defecto
    location / {
        try_files $uri $uri/ =404;
    }

    # Si necesitas PHP para el sitio por defecto:
    # index index.php index.html;
    # location / {
    #     try_files $uri $uri/ /index.php?$query_string;
    # }
    #
    # # Configuración PHP-FPM (si aplica para el default)
    # location ~ \.php$ {
    #     include snippets/fastcgi-php.conf;
    #     fastcgi_pass unix:/var/run/php/phpX.Y-fpm.sock; # Ajusta tu versión de PHP
    # }
}

# --- NOTA IMPORTANTE PARA SITIOS ADICIONALES ---
# Para cada sitio web adicional (ej: tusitioweb1.com, tusitioweb2.com, etc.),
# necesitarás DOS bloques `server`: uno para redirigir `www` y otro para el contenido no-`www`.
# El `root` en el segundo bloque debe coincidir con el `Path`
# que definiste en el módulo "Domains" de Easypanel (ej: /code/tusitioweb1).

# --- Servidor para tusitioweb1.com (Ejemplo Genérico) ---
server {
    listen 80;
    server_name www.tusitioweb1.com;
    return 301 $scheme://tusitioweb1.com$request_uri;
}
server {
    listen 80;
    server_name tusitioweb1.com www.tusitioweb1.com;
    root /code/tusitioweb1;
    index index.html;
    location / { try_files $uri $uri/ =404; }
}

# --- Servidor para tusitioweb2.com (Ejemplo Genérico) ---
server {
    listen 80;
    server_name www.tusitioweb2.com;
    return 301 $scheme://tusitioweb2.com$request_uri;
}
server {
    listen 80;
    server_name tusitioweb2.com www.tusitioweb2.com;
    root /code/tusitioweb2;
    index index.html;
    location / { try_files $uri $uri/ =404; }
}

# Repite la estructura de los dos bloques anteriores (Bloque X.1 y Bloque X.2)
# para cada sitio web adicional que quieras configurar.
```

Haz clic en **"Save"** o **"Guardar"**.

### 3. SFTP: SUBIDA DE ARCHIVOS DE NUESTROS SITIOS

a. Conéctate a tu servidor mediante SFTP.

b. Navega a la ruta de tu servicio:
   `/etc/easypanel/projects/NOMBRE_DEL_PROYECTO/services/NOMBRE_DEL_SERVICIO/code`
   *(Ej: `/etc/easypanel/projects/webs/services/static-website/code`)*

c. Dentro de `/code`, crea las carpetas para cada dominio (ej: `tusitioweb1`, `tusitioweb2`).

d. Sube los archivos de cada sitio a su carpeta correspondiente.

### 4. AJUSTAR PERMISOS (VÍA SSH): UN PASO IMPORTANTE

a. Conéctate a tu servidor por SSH.

b. Navega a la carpeta `code` de tu servicio:
   `cd /etc/easypanel/projects/webs/services/static-website/code` *(ajusta tu ruta)*

c. Cambia el propietario de las carpetas de tus sitios. En el video se usa `www-data`:
   ```bash
   sudo chown -R www-data:www-data tusitioweb1
   sudo chown -R www-data:www-data tusitioweb2
   ```
   *(Reemplaza `tusitioweb1` y `tusitioweb2` con los nombres de tus carpetas).*

### 5. RECONSTRUIR IMAGEN DOCKER Y ARRANCAR EL SERVICIO

a. En Easypanel, para tu servicio (ej: `static-website`), haz clic en **"Rebuild"**.

b. Una vez completado, haz clic en **"Start"**.

### 6. VERIFICAR FUNCIONAMIENTO

a. Comprueba los logs del servicio en Easypanel.

b. Accede a `https://tusitioweb1.com` (y `www.tusitioweb1.com`), y a `https://tusitioweb2.com` (y `www.tusitioweb2.com`), etc., en tu navegador.
nota: auso que tienes el tema de los certificados ssl resueltos, yo uso cloudflare o en su defecto easypanel usa Let's Encrypt.
En tus dns debe haber un registro cname www apuntando a tudominio1.com

c. La versión `www` debe redirigir a la no-`www`.

d. Si hay errores 404, asegúrate de que los archivos están en el `root` correcto y los permisos son correctos. Si persiste, revisa los logs de Nginx.

**FIN DE LAS INSTRUCCIONES**

Nota: si has llegado hasta aquí aun no he subido los dos videos de migrar en wordpress, si te interesan hazme saber en los comentarios.
