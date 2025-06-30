---
layout: default
nav_order: 2
permalink: /how-to-install-n8n-using-google-cloud
title: "Cómo instalar N8N GRATIS usando Google Cloud y Cloudflare"
lang: es
---
<style>
/*BEM (Block, Element, Modifier) methodology*/
.button {      
  border-radius: 4px;
  padding: 2px 6px;
  border: 1px solid #1a73e8;
}

.button--solid-blue {
    background-color: #1a73e8;
    color: #ffffff;    
}

.button--outline-blue {
  background-color: #ffffff;
  color: #1a73e8;  
}
</style>

# Cómo instalar N8N GRATIS Usando Google Cloud Y Cloudflare

Instale N8N gratis en su propio dominio (por ejemplo, `n8n.midominio.com`) aprovechando la Capa Gratuita de Google Cloud y el túnel administrado remotamente de Cloudflare. Este enfoque proporciona una instancia de N8N lista para producción de forma segura y sin costo.

## Prerrequisitos

- Regístrese en Google Cloud Platform ([console.cloud.google.com](https://console.cloud.google.com))
- Configure un Proyecto de Google Cloud ([console.cloud.google.com/projectcreate](https://console.cloud.google.com/projectcreate))
- Configure una Cuenta de Facturación de Cloud ([console.cloud.google.com/billing/create](https://console.cloud.google.com/billing/create))
- Vincule la Cuenta de Facturación al Proyecto ([console.cloud.google.com/billing/projects](https://console.cloud.google.com/billing/projects))
- Agregue su dominio a Cloudflare ([Agregar sitio a Cloudflare](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/#add-site-to-cloudflare) y [su registrador de dominio](https://www.youtube.com/watch?v=uqlo3lCqiy0))
- Regístrese en el plan gratuito Cloudflare Zero Trust ([one.dash.cloudflare.com](https://one.dash.cloudflare.com/))

## Paso 1: Cree una instancia de VM de Google Compute Engine

- Haga clic en <button class="button button--solid-blue">Crear instancia</button> ([Instancias de VM ↗](https://console.cloud.google.com/compute/instances)), habilite la API de Compute Engine (si se le solicita) y configure:
- **Configuración de la máquina**
    - Nombre: `n8n-free-tier` (o su nombre preferido)
    - Región: `us-west1 (Oregón)` (o su región preferida de: *us-west1 (Oregon)*, *us-central1 (Iowa)*, *us-east1 (South Carolina)*)
    - Tipo de máquina: `e2-micro`
- **Sistema operativo y almacenamiento** (SO y almacenamiento)
    - Haga clic en <button class="button button--solid-blue">Cambiar</button> y configure:
        - Sistema operativo: `Container Optimized OS`
        - Tipo de disco de arranque: `Standard persistent disk`
        - Tamaño (GB): `30`
    - Haga clic en <button class="button button--solid-blue">Seleccionar</button>
- **Protección de datos**
    - Seleccione `No backups` (Sin copias de seguridad)
- **Redes**
    - Active su interfaz de red (Network interfaces)
        - Cambie de Premium a `Standard` (Nivel de servicio de red)
- **Observabilidad**
    - Desmarque `Install Ops Agent for Monitoring and Logging` (Instalar Ops Agent para monitoreo y registro)
- Haga clic en <button class="button button--solid-blue">Crear</button>

## Paso 2: Cree un túnel de Cloudflare

- Haga clic en <button class="button button--solid-blue">Crear un túnel</button> ([Zero Trust ↗](https://one.dash.cloudflare.com/) > Redes > Túneles)
- Haga clic en <button class="button button--outline-blue">Seleccionar Cloudflared</button>
- Asigne un nombre a su túnel (por ejemplo, `n8n-gcp-tunnel`)
- Haga clic en <button class="button button--solid-blue">Guardar túnel</button>
- Haga clic en <button class="button button--solid-blue">Docker</button>
- Copie el valor del token (la cadena alfanumérica inmediatamente después de --token) del comando mostrado.
- Agregue el nombre de host público para su túnel
    - **Subdominio:** (por ejemplo, `n8n`)
    - **Dominio:** (por ejemplo, `midominio.com`)
    - **Tipo:** `HTTP`
    - **URL:** `n8n:5678`
- Haga clic en <button class="button button--solid-blue">Completar configuración</button>

## Paso 3: Instale n8n usando Docker

- Conéctese por `SSH` a su VM de Google Cloud ([Consola de Google Cloud ↗](console.cloud.google.com) > Compute Engine > Instancias de VM > Conectar)
- Cree un directorio
    ```bash
    mkdir ~/n8n-cloudflare
    cd ~/n8n-cloudflare
    ```
- Cree el archivo .env
    ```bash
    nano .env
    ```
- Pegue el contenido de la sección del archivo .env a continuación. Recuerde reemplazar los valores de marcador de posición para `CLOUDFLARE_TUNNEL_TOKEN` y `N8N_HOST`. Guarde y salga (Ctrl+X, Y, Enter).
    ```bash
    # .env file

    # Cloudflare Tunnel Token - REEMPLACE ESTO CON SU TOKEN REAL
    # Lo obtuvo al configurar el túnel en el panel de Cloudflare Zero Trust.
    # IMPORTANTE: ¡Mantenga este archivo seguro y NO lo suba al control de versiones!
    CLOUDFLARE_TUNNEL_TOKEN="eyJhIj..."

    # Configuración de N8N
    # El dominio principal donde n8n será accesible.
    N8N_HOST=n8n.mydomain.com

    # Zona horaria para n8n. Encuentre su cadena de zona horaria aquí: [https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
    GENERIC_TIMEZONE=America/New_York

    # Autenticación básica de N8N (MUY RECOMENDADO PARA LA SEGURIDAD)
    # Establezca en 'true' para habilitar la autenticación básica.
    N8N_BASIC_AUTH_ACTIVE=true
    ```
- Establezca permisos para .env
    ```bash
    chmod 600 .env
    ```
- Cree el archivo docker-compose.yml
    ```bash
    nano docker-compose.yml
    ```
- Pegue el contenido de la sección docker-compose.yml anterior. Guarde y salga
    ```yaml
    # docker-compose.yml

    version: '3.8' # Use a recent Docker Compose file format version

    networks:
      n8n_network:
        # A custom bridge network for our services to communicate securely
        # and isolate them from the host network directly.
        driver: bridge

    volumes:
      n8n_data:
        # This volume will store all of n8n's persistent data (workflows, credentials, configurations).
        # Data will persist even if the n8n container is re-created or updated.

    services:
      n8n:
        image: n8nio/n8n:latest # Use the official n8n image
        container_name: n8n # A friendly name for your container
        restart: unless-stopped # Always restart unless explicitly stopped

        # Internal port mapping for n8n.
        # n8n inside the container listens on 5678.
        # We are NOT exposing this port to the host machine directly (-p option),
        # because Cloudflare Tunnel will connect directly to it via the internal network.
        # For local testing or if you ever needed direct host access, you'd add:
        # ports:
        #   - "5678:5678" # This is NOT needed with Cloudflare Tunnel

        environment:
          # Load variables from the .env file
          - N8N_HOST=${N8N_HOST}
          - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
          - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}

          # Critical: Tell n8n its external URL for webhooks to work correctly
          # Cloudflare Tunnel will forward requests from [https://n8n.yourdomain.com](https://n8n.yourdomain.com)
          # to n8n's internal HTTP port. Cloudflare handles the HTTPS termination.
          # - WEBHOOK_URL=https://${N8N_HOST}/

          # For production, it's recommended to disable the editor if only webhooks are used.
          # You can re-enable it if you need to access the UI to create/manage workflows.
          # - N8N_EDITOR_DISABLED=true

          # Uncomment and configure if you plan to use an external database (e.g., PostgreSQL)
          # - DB_TYPE=postgresdb
          # - DB_POSTGRESDB_HOST=your_postgres_host
          # - DB_POSTGRESDB_PORT=5432
          # - DB_POSTGRESDB_DATABASE=n8n_db
          # - DB_POSTGRESDB_USER=n8n_user
          # - DB_POSTGRESDB_PASSWORD=your_postgres_password

        volumes:
          - n8n_data:/home/node/.n8n # Mount the named volume for persistent data

        networks:
          - n8n_network # Connect n8n to our custom network

      cloudflared:
        image: cloudflare/cloudflared:latest # Official Cloudflare Tunnel image
        container_name: cloudflared
        restart: unless-stopped # Always restart unless explicitly stopped

        environment:
          # Load the tunnel token from the .env file
          - TUNNEL_TOKEN=${CLOUDFLARE_TUNNEL_TOKEN}

        command: tunnel --no-autoupdate run # Standard command to run the tunnel

        # Crucial: Connect cloudflared to the same network as n8n so it can reach it.
        networks:
          - n8n_network

        # Note: We don't expose any ports on the host for cloudflared.
        # It establishes an outbound connection to Cloudflare's network.
    ```
- Inicie los servicios
    ```bash
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v "$(pwd):/app" -w /app docker/compose:latest up -d
    ```
Ahora, cuando acceda a https://n8n.mydomain.com.com en su navegador, Cloudflare enrutará el tráfico a través de su red, a través del túnel, directamente a su contenedor n8n, sin exponer la dirección IP de su VM.

## Limpieza

Próximamente: Instrucciones detalladas sobre cómo eliminar de forma fácil y completa su configuración de N8N y los recursos asociados de Google Cloud.

## Ventajas

- **Sin tráfico entrante:** No es necesario marcar `Permitir tráfico HTTPS` en las reglas de firewall, lo que evita la exposición directa de su VM y sus valiosos datos a la Internet pública.
- **HTTPS sin esfuerzo:** Deje de preocuparse por instalar y administrar certificados SSL. Cloudflare automatiza toda la terminación HTTPS por usted. No se necesitan herramientas como Nginx o Certbot.
- **Libertad de IP dinámica:** Olvídese de las direcciones IP estáticas o la gestión compleja de DNS. Su VM establece una conexión persistente, solo de salida, a la red perimetral de Cloudflare, lo que permite que la dirección IP de su VM cambie mientras su dominio N8N permanece accesible. Puede reiniciar o detener su instancia con confianza en cualquier momento, sin preocuparse por los cambios de dirección IP.
- **Rendimiento optimizado para contenedores:** Su instancia N8N se ejecuta en el sistema operativo optimizado para contenedores de Google (Container-Optimized OS), un sistema operativo especializado creado desde cero para ejecutar aplicaciones contenidas de manera eficiente y confiable, superando a las alternativas de uso común como Debian.
- **Nivel empresarial sin costo:** Aproveche la capa gratuita de Google Cloud y la red de grado empresarial de Cloudflare para implementar una instancia N8N robusta y lista para producción sin costo alguno.

## Limitaciones

- Una instancia de VM e2-micro no preemptiva por mes en regiones específicas de EE. UU. (Oregón: us-west1, Iowa: us-central1, Carolina del Sur: us-east1)
- 30 GB de almacenamiento en disco persistente estándar
- 200 GB/mes de salida de red gratuita
- Sin registro
- Sin monitoreo
- Sin programación de instantáneas

## Solución de problemas y problemas comunes

Próximamente: Soluciones a problemas comunes que pueda encontrar, junto con útiles consejos de solución de problemas para que vuelva a encarrilarse rápidamente.

## Mejoras adicionales

- Agregue un nombre de host de webhook separado
- Utilice una base de datos externa (por ejemplo, PostgreSQL)

## Recursos

- [Free Tier features \| Google Cloud](https://cloud.google.com/free/docs/free-cloud-features#free-tier)
- [Container-Optimized OS \| Google Cloud](https://youtu.be/jh0fPT-AWwM?t=211)
- [Container-Optimized OS Overview \| Google Cloud](https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits)
- [Create a tunnel (dashboard) \| Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
- [Hosting n8n on Google Cloud \| n8n Docs](https://docs.n8n.io/hosting/installation/server-setups/google-cloud/)
- [Self-Hosting n8n with Docker & Cloudflare Tunnel \| n8n Community](https://community.n8n.io/t/securely-self-hosting-n8n-with-docker-cloudflare-tunnel-the-arguably-less-painful-way/93801)
- [GCP Network Service Tiers (Cloud Performance Atlas)](https://www.youtube.com/watch?v=wsdgWGE-mwE)
- [Cloudflare origin CA](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)