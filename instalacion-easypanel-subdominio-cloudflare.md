# Guía: Cómo Instalar EasyPanel con un Dominio Personalizado

Esta guía te enseñará a instalar el panel de control **EasyPanel** en tu propio servidor y a configurarlo para que sea accesible desde un subdominio personalizado y seguro, como `panel.tudominio.com`.

El objetivo es tener una base sólida para gestionar todas tus aplicaciones auto-alojadas (self-hosted) de forma sencilla y profesional.

Por: Guillermo Gutiérrez [https://www.linkedin.com/in/ggd79/](https://www.linkedin.com/in/ggd79/)

### **Requisitos Previos**

* **Un servidor VPS** con una versión reciente de Ubuntu.
* **Un dominio** ya comprado y registrado.
* **Una cuenta gratuita en Cloudflare**, con tu dominio ya añadido. (opcional)

---

### **Paso 1: Instalación de EasyPanel en el Servidor**

Primero, instalaremos el panel en tu máquina virtual.

1.  **Conéctate a tu servidor por SSH.**
2.  **Ejecuta el comando de instalación oficial.** El script se encargará de todo.
    ```bash
    curl -sSL https://get.easypanel.io | sh
    ```
3.  Cuando termine, **accede al panel** desde un navegador usando la IP de tu servidor y el puerto `3000`.
    `http://<tu_ip_de_servidor>:3000`
4.  **Crea tu cuenta de administrador.** Esta será tu cuenta principal para gestionar el panel.

---

### **Paso 2: Apuntar tu Subdominio con Cloudflare**

Ahora, le diremos a Cloudflare que el subdominio `panel.tudominio.com` debe apuntar a tu servidor.

1.  **Inicia sesión en Cloudflare** y selecciona tu dominio.
2.  Ve a la sección **DNS > Records**.
3.  **Añade un registro de tipo `A`** para el panel. Reemplaza `<tu_ip_de_servidor>` con la IP de tu VPS.
    * **Tipo:** `A`
    * **Nombre:** `panel` (o el que prefieras: `app`, `cloud`, etc.)
    * **Contenido:** `<tu_ip_de_servidor>`
4.  **Verificación Importante:** Asegúrate de que la nube de Cloudflare esté de color **naranja (modo "Proxy" activado)**. Esto te da SSL gratis y oculta tu IP.
5.  **Configura el modo SSL/TLS:**
    * Ve a **SSL/TLS > Visión General**.
    * Selecciona el modo **"Completo (estricto)"** para la máxima seguridad.

![cloudflare](https://github.com/guille1one/guias-web/blob/main/img/cloudflare_ssl.png?raw=true)

---

### **Paso 3: Vincula el Dominio en EasyPanel**

Este es el paso final para lograr nuestro objetivo.

1.  Vuelve a la interfaz de EasyPanel (todavía usando la IP).
2.  Ve a **Settings > General**.
3.  En el campo "Custom Domain", introduce el subdominio que configuraste en Cloudflare: `panel.tudominio.com`.
4.  Guarda los cambios.

¡Listo! A partir de ahora, ya puedes **acceder a tu panel de control desde `https://panel.tudominio.com`**. Tu conexión es segura y la URL es profesional.

---

### **¿Y ahora qué? Ejemplo Rápido: Instalar n8n**

Ahora que tienes tu panel, instalar cualquier aplicación es muy fácil. Aquí tienes un ejemplo con n8n:

1.  **DNS en Cloudflare:** Crea otro registro `A` para `n8n` apuntando a la misma IP (con el proxy naranja activado).
2.  **Instalar desde Plantilla:** En EasyPanel, ve a `Templates`, busca `n8n` y haz clic en `Install`.
3.  **Asignar Dominio:**
    * Dentro del nuevo proyecto n8n, ve a la pestaña `Domains`.
    * Añade el host `n8n.tudominio.com` y el puerto `5678`.
    * Márcalo como principal con el icono de la estrella (⭐).
4.  **Configurar Webhook (Crítico para n8n):**
    * Ve a la pestaña `Environment`.
    * Añade la variable `WEBHOOK_URL` con el valor `https://n8n.tudominio.com/`.
    * Haz clic en **"Redeploy"** para aplicar los cambios.

Ahora `n8n.tudominio.com` está funcionando. Puedes seguir este mismo proceso para instalar WordPress, Ghost, bases de datos y mucho más.
