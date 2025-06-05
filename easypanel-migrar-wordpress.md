# Migración Manual de wordpress a EasyPanel
_por Guillermo Gutiérrez_

## 1. EasyPanel

* Añadir servicio **WordPress** y apagar.
* Asignar **Límites de Recursos** (a tu criterio):
    * Memoria `0 256`
    * CPU `0 0.5`
* Añadir **Dominio**:
    * `tusitioweb.com`

## 2. Hosting (Antiguo)

* Entrar a **phpMyAdmin** y exportar dy escargar el archivo `.sql` de la base de datos.
* Desde el explorador de archivos, comprimir `wp-content` a `.tar.gz`.

---

## EasyPanel - Transferencia y Configuración

### Base de Datos

* Abrir el archivo `.sql` con un editor.
* Cambiar las rutas de directorios locales por `/var/www/html`.
    * **Nota:** Dependiendo del hosting, la ruta podría empezar por `/home/(algún código identificador)/public_html...`. Por ejemplo: `/home/1234/public_html/sitioweb/`. Ten cuidado al seleccionar hasta el final (con o sin `/`), ya que esto podría ocasionar un error. Por ejemplo: `(/home/1234/public_html/sitioweb)`.
* En caso de tener un solo servicio de base de datos para distintas webs:
    * En **phpMyAdmin**, seleccionar la base de datos vacía.
    * En **phpMyAdmin**, importar el archivo `.sql`.

### Archivos (Terminal VPS)

* Navegar a la ruta del volumen del sitio. Por lo general, se encuentra en `/etc/easypanel/projects/nombredetuproyecto`.
* Subir el archivo por SFTP o copiar directamente con el comando: `wget tusitioweb.com/wp-content.tar.gz`.
* Descomprimir `wp-content.tar.gz` usando el comando: `tar -xzvf wp-content.tar.gz`.
* Eliminar `wp-content.tar.gz` con el comando: `rm wp-content.tar.gz`.

### Permisos (Terminal VPS)

* Aplicar `chown` y `chmod` para `wp-content` (o para todo el directorio `data/`).
* Dentro de la carpeta de tu proyecto (`/volumes/data`), aplicar el comando: `chown -R www-data:www-data wp-content/`.

## 3. Pasos Finales

* Cambiar la IP del dominio. (en algunos casos puede tardar entre 24 y 48 horas )
* Probar la URL de la página hasta que aparezca la pantalla de **EasyPanel**. (esto indica que el dominio ya esta apuntando al nuevo servidor)
* Arrancar el contenedor **WordPress**.
* Probar el sitio.

---

### Dentro de WordPress

* Regenerar los **Permalinks** en caso de algún problema con los enlaces.

---

# Recordatorios Importantes

* **Reiniciar Contenedores:** Después de establecer o modificar estas variables de entorno para cada servicio de **WordPress**, es crucial que reinicies el contenedor/servicio de **WordPress** correspondiente para que tome la nueva configuración.
* **Permisos en MariaDB:** Asegúrate de que cada usuario tenga `ALL PRIVILEGES` sobre su base de datos respectiva y que esté permitido conectarse desde el host o la IP interna de tus contenedores de **WordPress**.

---

## BONUS:

Si usas un solo servicio de base de datos para varios **WordPress**, te dejo la plantilla para crear la base de datos y asignar un usuario para cada una. Luego, debes ir a las variables de entorno del contenedor de **WordPress** y cambiar el usuario, el nombre de la base de datos y la contraseña.

**SQL que ejecutas en phpMyAdmin:**

```sql
-- #####################################################################
-- ## Base de Datos: EjemploGuillermo
-- #####################################################################
CREATE DATABASE IF NOT EXISTS `ejemploguille` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'guille_usr'@'localhost' IDENTIFIED BY 'TuC0nTr4S3ñA'; -- Contraseña ya establecida
GRANT ALL PRIVILEGES ON `ejemploguille`.* TO 'guille_usr'@'localhost';

CREATE USER IF NOT EXISTS 'guille_usr'@'10.11.%.%' IDENTIFIED BY 'TuC0nTr4S3ñA'; -- Contraseña ya establecida
GRANT ALL PRIVILEGES ON `ejemploguille`.* TO 'guille_usr'@'10.11.%.%';
```

Este bloque de SQL:
1.  Crea una base de datos llamada `ejemploguille` si no existe, con juego de caracteres `utf8mb4`.
2.  Crea un usuario `guille_usr` para conexiones desde `localhost` con la contraseña `TuC0nTr4S3ñA` (que ya estaría establecida según el comentario).
3.  Otorga todos los privilegios sobre la base de datos `ejemploguille` a este usuario `guille_usr` cuando se conecta desde `localhost`.
4.  Crea el mismo usuario `guille_usr` para conexiones desde el rango de IP `10.11.%.%` (esto es útil para redes internas de contenedores, por ejemplo) con la misma contraseña.
5.  Otorga todos los privilegios sobre la base de datos `ejemploguille` a este usuario `guille_usr` cuando se conecta desde ese rango de IP.

## ¡Conecta Conmigo! 🔗

Si esta guía te ha sido de ayuda o quieres explorar más contenido y proyectos, ¡me encantaría conectar contigo! Puedes encontrarme en:

* **LinkedIn:** [Guillermo Gutiérrez](https://www.linkedin.com/in/ggd79/)

* **YouTube:** [Canal de Guillermo Gutiérrez](https://www.youtube.com/@gg1one)

¡Espero que estas guías y videos te sean de gran utilidad!
