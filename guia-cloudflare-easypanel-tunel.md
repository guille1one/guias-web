# Guía Definitiva: Zero Trust con EasyPanel (Túneles + Acceso)

Esta guía describe el proceso para securizar un servidor autogestionado con EasyPanel utilizando la arquitectura de **Zero Trust de Cloudflare**. El objetivo es transformar un servidor estándar en una **fortaleza digital impenetrable**, haciéndolo **100% invisible** a Internet y permitiendo el acceso a las aplicaciones únicamente a usuarios autorizados.

### Arquitectura Final

El resultado de esta guía será una configuración donde todo el tráfico es gestionado a través de un túnel seguro y encriptado, eliminando la exposición directa del servidor a Internet.

![Diagrama de una conexión segura con Cloudflare Tunnel](https://github.com/guille1one/guias-web/blob/main/img/zero-trust.webp)
---

### **Etapa 1: Preparación - El Túnel y la Conexión**

El túnel es el pasadizo secreto y encriptado entre tu servidor y Cloudflare.

#### **1. El "Acto de Fe": Eliminar Rutas Públicas**

* Si partes de una configuración existente con registros DNS públicos, es crucial eliminarlos. Ve a Cloudflare -> DNS -> Records y **borra los registros `A`** de tus aplicaciones (ej: `panel`, `n8n`, etc.). Si empiezas de cero, simplemente no los crees.

#### **2. Obtener el API Token de Cloudflare (Opcional)**

* **Nota:** Este token solo es necesario si se desea usar la integración automática de EasyPanel. Para el método manual descrito en esta guía, no es necesario.

#### **3. Crear el Túnel en Cloudflare**

* En el menú de Cloudflare, ve a **Zero Trust**. La primera vez, te pedirá un "team name" (elige uno y selecciona el plan gratuito).
* Navega a **Networks -> Tunnels** y haz clic en **"Create a tunnel"**.
* Dale un nombre (ej: `easypanel-servidor`) y guárdalo.
* Copia el comando largo que aparece en la siguiente pantalla (ese es el token de tu túnel).
* Conéctate a tu servidor por SSH, pega y ejecuta el comando. Esto instalará y activará `cloudflared` como un servicio.
* Verifica que el túnel aparezca como **"HEALTHY"** en Cloudflare.

---

### **Etapa 2: La Estrategia de los 3 Subdominios (La Clave del Éxito)**

Para lograr la máxima seguridad con la mínima complejidad, asignaremos una "puerta" única para cada función. Necesitaremos tres subdominios en total:

1.  **Puerta de Gerencia:** Para la UI de EasyPanel (ej: `panel.tudominio.com`).
2.  **Puerta de Empleados:** Para la UI de n8n (ej: `n8ndev.tudominio.com`).
3.  **Muelle de Carga:** Exclusivamente para los Webhooks de n8n (ej: `automatize.tudominio.com`).

#### **1. Configurar los "Public Hostnames" en tu Túnel**

* Dentro de la configuración de tu túnel, en la pestaña "Public Hostname", añade las tres rutas:
    * `panel.tudominio.com` -> `easypanel:3000`
    * `n8ndev.tudominio.com` (para la UI) -> `dev_n8n:5678`
    * `automatize.tudominio.com` (para Webhooks) -> `dev_n8n:5678`

#### **2. Actualizar n8n**

* Ve a EasyPanel, a tu proyecto n8n -> `Environment`.
* Cambia la variable `WEBHOOK_URL` a tu subdominio de "Muelle de Carga": `https://automatize.tudominio.com/`
* Guarda y haz **"Redeploy"**.

---

### **Etapa 3: Control de Acceso (El Guardia de Seguridad)**

Ahora que tenemos las puertas definidas, pondremos un guardia solo en las que lo necesitan.

1.  **Crear la Política de Acceso:** En Zero Trust, ve a **Access -> Policies**. Crea una política `Admin Access` con `Action: Allow` que incluya tu correo electrónico.
2.  **Proteger las Aplicaciones:** Ve a **Access -> Applications** y crea una aplicación "Self-hosted" **solo para las interfaces de usuario**:
    * **Para EasyPanel:** Protege `panel.tudominio.com` y asígnale la política `Admin Access`.
    * **Para la UI de n8n:** Protege `n8ndev.tudominio.com` y asígnale la política `Admin Access`.

> **Nota Clave:** No se ha creado una aplicación para el subdominio de webhooks (`automatize.tudominio.com`). Al no tener una política de acceso, el tráfico pasa por el túnel seguro pero sin pedir login. No se necesitan reglas de firewall. Esta es la arquitectura Zero Trust en su forma más pura y robusta.

---

### **Etapa 4: Securización Final**

Ahora que todo funciona, sellamos la fortaleza.

1.  **Desactivar Acceso por IP en EasyPanel:** En `Settings`, desactiva **"Serve on IP address"**. Esto previene el acceso directo a las aplicaciones a través de la dirección IP pública del servidor.

**Resultado:** El servidor está ahora protegido con una arquitectura Zero Trust limpia, profesional y fácil de mantener.
