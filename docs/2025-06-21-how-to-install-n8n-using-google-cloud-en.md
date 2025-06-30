---
layout: default
nav_order: 2
permalink: /how-to-install-n8n-using-google-cloud
title: "How To Install N8N FREE Using Google Cloud And Cloudflare"
lang: en
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

# How To Install N8N FREE Using Google Cloud And Cloudflare

 Install N8N for free on your own domain (e.g., `n8n.mydomain.com`) by leveraging the Google Cloud Free Tier and Cloudflare Remotely-managed tunnel. This approach provides a production-ready N8N instance in a secure way at no cost.

## Prerequisites

- Sign Up for Google Cloud Platform ([console.cloud.google.com](https://console.cloud.google.com))

- Set Up a Google Cloud Project ([console.cloud.google.com/projectcreate](https://console.cloud.google.com/projectcreate))

- Configure a Cloud Billing Account ([console.cloud.google.com/billing/create](https://console.cloud.google.com/billing/create))

- Link Billing Account to Project ([console.cloud.google.com/billing/projects](https://console.cloud.google.com/billing/projects))

- Add your domain to Cloudflare ([Add site to Cloudflare](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/#add-site-to-cloudflare) and [your domain registrar](https://www.youtube.com/watch?v=uqlo3lCqiy0))

- Sign Up for Cloudflare Zero Trust Free plan ([one.dash.cloudflare.com](https://one.dash.cloudflare.com/))

## Step 1: Create a Google Compute Engine VM instance
- Click <button class="button button--solid-blue">Create Instance</button> ([VM Instances ↗](https://console.cloud.google.com/compute/instances)), enable Compute Engine API (if asked) and configure:
- **Machine configuration**
    - Name: `n8n-free-tier` (or your preferred name)
    - Region: `us-west1 (Oregon)` (or your preferred region from: us-west1 (Oregon) | us-central1 (Iowa) | us-east1 (South Carolina))
    - Machine Type: `e2-micro`
- **Operating system and storage** (OS and storage) 
    - Click <button class="button button--solid-blue">Change</button> and configure:
        - Operating system: `Container Optimized OS`
        - Boot disk type: `Standard persistent disk`
        - Size (GB): `30`
    - Click <button class="button button--solid-blue">Select</button>
- **Data protection**
    - Select `No backups`
- **Networking**    
    - Toggle your network interface (Network interfaces)
        - Switch from Premium to `Standard` (Network Service Tier)
- **Observability**
    - Uncheck `Install Ops Agent for Monitoring and Logging`
- Click <button class="button button--solid-blue">Create</button>

## Step 2: Create a Cloudflare Tunnel

- Click <button class="button button--solid-blue">Create a tunnel</button> ([Zero Trust ↗](https://one.dash.cloudflare.com/) > Networks > Tunnels)
- Click <button class="button button--outline-blue">Select Cloudflared</button>
- Name your tunnel (e.g., `n8n-gcp-tunnel`)
- Click <button class="button button--solid-blue">Save tunnel</button>
- Click <button class="button button--solid-blue">Docker</button>
- Copy the token value (the alphanumeric string immediately after --token) from the displayed command.
- Add public hostname your for your tunnel
    - **Subdomain:** (e.g., `n8n`)
    - **Domain:** (e.g., `mydomain.com`)
    - **Type:** `HTTP`
    - **URL:** `n8n:5678`
- Click <button class="button button--solid-blue">Complete Setup</button>


## Step 3: Install n8n using Docker

- `SSH` into your Google Cloud VM ([Google Cloud console ↗](console.cloud.google.com) > Compute Engine > VM instances > Connect)
- Create a directory  
    ```bash
    mkdir ~/n8n-cloudflare
    cd ~/n8n-cloudflare
    ```
- Create the .env file
    ```bash
    nano .env
    ```
- Paste the content from the .env file section below. Remember to replace the placeholder values for `CLOUDFLARE_TUNNEL_TOKEN` and `N8N_HOST`. Save and exit (Ctrl+X, Y, Enter).
    ```bash
    # .env file

    # Cloudflare Tunnel Token - REPLACE THIS WITH YOUR ACTUAL TOKEN
    # You got this when setting up the tunnel in the Cloudflare Zero Trust dashboard.
    # IMPORTANT: Keep this file secure and do NOT commit it to version control!
    CLOUDFLARE_TUNNEL_TOKEN="eyJhIj..."

    # N8N Configuration
    # The primary domain where n8n will be accessible.
    N8N_HOST=n8n.mydomain.com

    # Timezone for n8n. Find your timezone string here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
    GENERIC_TIMEZONE=America/New_York

    # N8N Basic Authentication (HIGHLY RECOMMENDED FOR SECURITY)
    # Set to 'true' to enable basic authentication.
    N8N_BASIC_AUTH_ACTIVE=true
    ```
- Set permissions for .env
    ```bash
    chmod 600 .env
    ```
- Create the docker-compose.yml file
    ```bash
    nano docker-compose.yml
    ```
- Paste the content from the docker-compose.yml section above. Save and exit
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
          # Cloudflare Tunnel will forward requests from https://n8n.yourdomain.com
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
- Start the services
    ```bash
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v "$(pwd):/app" -w /app docker/compose:latest up -d
    ```
Now, when you access https://n8n.mydomain.com.com in your browser, Cloudflare will route the traffic through its network, via the tunnel, directly to your n8n container, without ever exposing your VM's IP address.

## Clean Up
Coming soon: Detailed instructions on how to easily and completely remove your N8N setup and associated resources from Google Cloud.

## Advantages

- **No Incoming Traffic At All:** No need to check `Allow HTTPS traffic` under the firewall rules, reventing direct exposure of your VM and its precious data to the public internet.

- **Effortless HTTPS:** Stop worrying about installing and managing SSL certificates. Cloudflare automates all HTTPS termination for you. No need for tools like Nginx or Certbot.

- **Dynamic IP Freedom:** Forget about static IP addresses or complex DNS management. Your VM establishes a persistent, outbound-only connection to Cloudflare's edge network, allowing your VM's IP address to change while your N8N domain remains accessible. You can confidently reboot or stop your instance at any time, without worrying about IP address changes.

- **Container-Optimized Performance:** Your N8N instance runs on Google's Container-Optimized OS, a specialized operating system built from the ground up for running contained applications efficiently and reliably, surpassing commonly used alternatives like Debian.

- **Enterprise Level at No Cost:** Leverage the Google Cloud Free Tier and Cloudflare's enterprise-grade network to deploy a robust, production-ready N8N instance at absolutely no cost.

## Limitations 
- One non-preemptible e2-micro VM instance per month in specific US regions (Oregon: us-west1, Iowa: us-central1, South Carolina: us-east1)
- 30 GB of standard persistent disk storage
- 200 GB / mo free network egress
- No Logging
- No Monitoring
- No Snapshot scheduling

## Troubleshooting & Common Issues
Coming soon: Solutions to common problems you might encounter, along with helpful troubleshooting tips to get you back on track quickly.
    
## Further Enhancements
- Add a separate webhook hostname
- Use an external database (e.g., PostgreSQL)   

## Resources
- [Free Tier features | Google Cloud](https://cloud.google.com/free/docs/free-cloud-features#free-tier)
- [Container-Optimized OS | Google Cloud](https://youtu.be/jh0fPT-AWwM?t=211)
- [Container-Optimized OS Overview | Google Cloud](https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits)
- [Create a tunnel (dashboard) | Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
- [Hosting n8n on Google Cloud | n8n Docs](https://docs.n8n.io/hosting/installation/server-setups/google-cloud/)
- [Self-Hosting n8n with Docker & Cloudflare Tunnel | n8n Community](https://community.n8n.io/t/securely-self-hosting-n8n-with-docker-cloudflare-tunnel-the-arguably-less-painful-way/93801)
- [GCP Network Service Tiers (Cloud Performance Atlas)](https://www.youtube.com/watch?v=wsdgWGE-mwE)
- [Cloudflare origin CA](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)





