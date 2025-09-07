# Guía Definitiva: Zero Trust con EasyPanel (Túneles + Acceso)

![ Zero Trust](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/zero-trust.webp)

Esta guía describe el proceso para securizar un servidor autogestionado con EasyPanel utilizando la arquitectura de **Zero Trust** de **Cloudflare**. El objetivo es transformar un servidor estándar en una **fortaleza digital impenetrable**, haciéndolo **100% invisible** a Internet y permitiendo el acceso a las aplicaciones únicamente a usuarios autorizados.

Por: Guillermo Gutiérrez [https://www.linkedin.com/in/ggd79/](https://www.linkedin.com/in/ggd79/)

### Nota Preliminar y Alcance de la Guía

Es importante entender el contexto de esta configuración. La arquitectura aquí descrita representa una mejora masiva en la seguridad de un servidor auto-alojado, especialmente contra ataques automatizados, escaneos de bots y actores maliciosos de nivel bajo a medio.

Esta guía es el resultado de mi propia exploración y conclusiones. No me considero un experto en seguridad, pero el objetivo es compartir una configuración que reduce significativamente los riesgos, enfocada en maximizar la seguridad utilizando las potentes herramientas que **Cloudflare** ofrece en su **plan gratuito**.

Sin embargo, hay que tener en cuenta:

* **La seguridad al 100% no existe.** Un atacante dedicado y con recursos avanzados podría encontrar otros vectores de ataque.
* Existen otras técnicas y herramientas más potentes, como el **WAF (Web Application Firewall) de pago** de Cloudflare, que ofrecen capas de protección adicionales.
* El objetivo de esta guía es construir la base más sólida posible con las herramientas gratuitas, lo cual ya supone una barrera de entrada muy significativa para la gran mayoría de amenazas.

### Arquitectura Final

El resultado de esta guía será una configuración donde todo el tráfico es gestionado a través de un túnel seguro y encriptado, eliminando la exposición directa del servidor a Internet.

### **Etapa 1: Preparación - El Túnel y la Conexión**

El túnel es el enlace seguro y encriptado que tu servidor establece con la red de Cloudflare.

#### **1. Eliminar Registros DNS Públicos**

* Si partes de una configuración existente con registros DNS públicos, es crucial eliminarlos. Ve a Cloudflare -> DNS -> Records y **borra los registros `A`** de tus aplicaciones (ej: `panel`, `n8n`, etc.). Si empiezas de cero, simplemente no los crees.

#### **2. Crear el API Token para la Integración (Requerido)**

* Para que EasyPanel pueda gestionar el túnel en tu nombre, necesita un API Token con los permisos adecuados. Este token actúa como una "llave" que le da permiso a EasyPanel para comunicarse con tu cuenta de Cloudflare.

 1.  Inicia sesión en tu cuenta de Cloudflare.
 2.  Ve a tu perfil -> **"API Tokens"** o directamente a `https://dash.cloudflare.com/profile/api-tokens`.
 3.  Haz clic en **"Create Token"** y luego selecciona **"Create Custom Token"**.
 4.  Dale un nombre descriptivo al token (ej: "EasyPanel Tunnel Token").
 5.  Añade los siguientes permisos, uno por uno:
     * `Account` -> `Account Settings` -> `Read`
     * `Account` -> `Cloudflare Tunnel` -> `Edit`
     * `Zone` -> `Zone` -> `Read`
     * `Zone` -> `DNS` -> `Edit`
 6.  Continúa al resumen (`Continue to summary`) y haz clic en **"Create Token"**.
 7.  **Copia el token generado y guárdalo en un lugar seguro.** Solo se mostrará una vez.
 
![token-permission](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/token-permission.webp)

#### **3. Crear el Túnel en Cloudflare y Conectarlo desde EasyPanel (Flujo Correcto)**

Para tener control total y evitar posibles errores, crearemos primero el túnel en Cloudflare y luego lo conectaremos desde EasyPanel.

1.  **En Cloudflare:**
    * Ve a **Zero Trust** -> `Networks` -> `Tunnels` y haz clic en **"Create a tunnel"**.
    * Dale un nombre descriptivo (ej: `easypanel-servidor`) y guárdalo.
    * En la siguiente pantalla, ignora los comandos de la consola. Ya has creado el túnel, eso es todo lo que necesitábamos aquí.

2.  **En EasyPanel:**
    * Ve a tu panel de EasyPanel -> `Settings`.
    * Pega el **API Token** que creaste en el paso anterior.
    * EasyPanel se conectará a tu cuenta. Ahora, en el desplegable de túneles, **selecciona el túnel que acabas de crear en Cloudflare** (`easypanel-servidor`).
    * Sigue los pasos que te indique la interfaz. EasyPanel usará el túnel seleccionado e instalará el agente `cloudflared` en el servidor por ti.

3.  **Verificación Final:** Ve a tu panel de Cloudflare. El túnel que creaste (`easypanel-servidor`) ahora debería tener el estado **"HEALTHY"**.

### **Etapa 2: Configuración de Hostnames (Rutas de Acceso)**

Para una seguridad óptima y simplicidad de gestión, configuraremos subdominios específicos para cada tipo de acceso a tus servicios. **El túnel por sí solo no sabe a dónde dirigir el tráfico; nosotros debemos crear estas reglas manualmente.**

1.  **Acceso a EasyPanel UI:** (ej: `panel.tudominio.com`).
2.  **Acceso a n8n UI:** (ej: `n8n.tudominio.com`).
3.  **Acceso a n8n Webhooks:** (ej: `n8nhook.tudominio.com`).

#### **1. Configurar los "Public Hostnames" en tu Túnel**

* Dentro de la configuración de tu túnel en Cloudflare Zero Trust, en la pestaña "Public Hostname", añade las siguientes rutas:

    > **Nota:** La URL del servicio interno en EasyPanel sigue el formato: `nombre-del-proyecto_nombre-de-la-aplicacion:puerto`. Por ejemplo, si tu proyecto se llama `dev` y tu aplicación n8n se llama `n8n`, la ruta interna será `dev_n8n:5678`.

    * `panel.tudominio.com` -> `easypanel:3000`
    * `n8n.tudominio.com` (para la UI de n8n) -> `dev_n8n:5678`
    * `n8nhook.tudominio.com` (para los Webhooks de n8n) -> `dev_n8n:5678`

#### **2. Actualizar la Variable de Entorno de n8n**

* Ve a EasyPanel, a tu proyecto n8n -> `Environment`.
* Cambia la variable `WEBHOOK_URL` a tu subdominio dedicado a webhooks: `https://n8nhook.tudominio.com/`
* Guarda los cambios y haz **"Redeploy"** de la aplicación n8n.

### **Etapa 3: Control de Acceso (Políticas Zero Trust)**

![/zero-trust-protected](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/zero-trust-protected.webp)

Ahora configuraremos las políticas para restringir el acceso a las interfaces de usuario.

1.  **Crear la Política de Acceso:** En Cloudflare Zero Trust, ve a **Access -> Policies**. Crea una política con el nombre `Admin Access`, establece la `Action: Allow`, y en las reglas de `Include`, añade tu dirección de correo electrónico.
2.  **Proteger las Aplicaciones:** Ve a **Access -> Applications** y crea una aplicación "Self-hosted" **solo para las interfaces de usuario**:

    * **Para EasyPanel:** Protege `panel.tudominio.com` y asígnale la política `Admin Access`.
    * **Para la UI de n8n:** Protege `n8n.tudominio.com` y asígnale la política `Admin Access`.
    * **Para otras aplicaciones (Ejemplo: Proteger el Super Admin de Chatwoot):**
        Este mismo proceso es especialmente útil para proteger rutas de administración específicas en lugar de todo un subdominio.

        1.  Asegúrate de haber añadido el hostname de Chatwoot en tu túnel (ej: `chat.tudominio.com -> chatwoot_app:3000`).
        2.  Crea una nueva aplicación de acceso para `chat.tudominio.com`.
        3.  En el campo **Path**, añade la ruta específica: `/super_admin`.
        4.  Asigna la política `Admin Access`.
            *Resultado:* `chat.tudominio.com` será público, pero el acceso a `/super_admin` requerirá autenticación.

> **Nota Importante:** No es necesario crear una aplicación de acceso para los subdominios destinados a webhooks (ej: `n8nhook.tudominio.com`). Esto permite que el tráfico destinado a estas URLs pase a través del túnel de forma segura y directa, sin requerir autenticación. Esto es fundamental para el funcionamiento de webhooks tanto en n8n como en otras aplicaciones, por ejemplo, la URL de webhook para una integración en Chatwoot.

### **Etapa 4: Endurecimiento de Webhooks con Reglas de Seguridad (Avanzado)**

Ahora que hemos bloqueado el acceso, daremos un paso adicional para endurecer la seguridad de la URL de n8n. El objetivo es permitir el paso de los webhooks que contienen un identificador único (ej: `/webhook/123-abc`), pero evitar que las rutas base (ej: `/webhook/`) puedan usarse para llegar a la interfaz de n8n. Esto se logra con dos reglas en un orden específico.

1.  Ve a tu dominio en Cloudflare -> **Security -> Security Rules**.
2.  **Regla 1: Permitir Rutas de Webhook Específicas**
    * **Rule name:** `Allow n8n Specific Paths`
    * **When incoming requests match...**
        Configura las siguientes condiciones, agrupándolas con el operador **OR**:

# Allow webhooks and forms

### When incoming requests match...

* **`OR`** `Hostname equals n8nhook.tudominio.com AND URI Path contains /webhook/ AND URI Path does not end with /webhook/`

* **`OR`** `Hostname equals n8nhook.tudominio.com AND URI Path contains /webhook-test/ AND URI Path does not end with /webhook-test/`

* **`OR`** `Hostname equals n8nhook.tudominio.com AND URI Path contains /form/ AND URI Path does not end with /form/`

* **`OR`** `Hostname equals n8nhook.tudominio.com AND URI Path contains /form-test/ AND URI Path does not end with /form-test/`
    
    * **Then... Action:** `Skip`
    * **Skip:** Marca la opción `Access Control (Access)`.
    * NOTA: puedes copiar y pegar esta expresion:
`(http.request.uri.path wildcard r"*/webhook/*" and not ends_with(http.request.uri.path, "webhook/")) or (http.request.uri.path wildcard r"*/webhook-test/*" and not ends_with(http.request.uri.path, "webhook-test/")) or (http.request.uri.path wildcard r"*/form/*" and not ends_with(http.request.uri.path, "form/")) or (http.request.uri.path wildcard r"*/form-test/*" and not http.request.uri.path contains "form-test/")`
  
![security-rules-allow](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/security-rules-allow.webp)

3.  **Regla 2: Bloquear Rutas de Webhook Base**
    * **Rule name:** `Block n8n Base Paths`
    * **When incoming requests match...** Usa el "Expression Editor" y pega la siguiente expresión (cambia el dominio por el tuyo):
        ```
        (http.host eq "n8nhook.tudominio.com" and http.request.uri.path in {"/webhook/" "/webhook-test/" "/form/"})
        ```
    * **Then... Action:** `Block`
![security-rules-block](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/security-rules-block.webp)

4.  **Verificar el Orden Final**
    * Es **CRÍTICO** que las reglas se ejecuten en el orden correcto. En la lista de Reglas de Seguridad, arrastra la regla **`Allow n8n Specific Paths`** para que quede **POR ENCIMA** de la regla **`Block n8n Base Paths`**.
    * El flujo será: Cloudflare primero comprueba si es un webhook válido y se salta el login. Si no lo es, comprueba si es una ruta base y la bloquea. Si no es ninguna de las dos (ej: la raíz "/"), la petición continúa y es interceptada por la Política de Acceso que creamos en la Etapa 3.

![security-rules](https://github.com/guille1one/guias-web/blob/main/img/zero-trust/security-rules.webp)

### **Etapa 5: Securización Final**

Para completar la configuración, aseguraremos que no haya otras vías de acceso al servidor.

1.  **Desactivar Acceso por IP en EasyPanel:** En la configuración de EasyPanel (`Settings`), desactiva la opción **"Serve on IP address"**. Esto previene el acceso directo a las interfaces de tus aplicaciones a través de la dirección IP pública del servidor.

**Resultado:** El servidor está ahora protegido con una arquitectura Zero Trust limpia, profesional y fácil de mantener, asegurando que solo el tráfico autorizado y gestionado por Cloudflare pueda alcanzar tus servicios.

### **Más Allá de esta Guía: Técnicas Avanzadas**

La seguridad es un campo en constante evolución. La configuración de esta guía es una base excelente, pero existen métodos aún más complejos y granulares para proteger tus servicios, especialmente si estás construyendo APIs o servicios que se comunican entre sí. Algunas de estas técnicas avanzadas incluyen el uso de **tokens de autenticación (Bearer Tokens) en las cabeceras de las peticiones a tus URLs de webhooks**, lo que añade una capa adicional de verificación a nivel de aplicación. Estas configuraciones quedan fuera del alcance de esta guía, pero son un siguiente paso lógico para quien quiera profundizar aún más en la seguridad de sus sistemas.
