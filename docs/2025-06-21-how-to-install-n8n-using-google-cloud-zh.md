---
layout: default
nav_order: 2
permalink: /how-to-install-n8n-using-google-cloud
title: "如何使用 Google Cloud 和 Cloudflare 免费安装 N8N"
lang: zh
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

# 如何使用 Google Cloud 和 Cloudflare 免费安装 N8N

利用 Google Cloud 免费套餐和 Cloudflare 远程管理隧道，在您自己的域（例如，`n8n.mydomain.com`) 上免费安装 N8N。此方法以安全、免费的方式提供生产就绪的 N8N 实例。

## 先决条件

- 注册 Google Cloud Platform ([console.cloud.google.com](https://console.cloud.google.com))
- 设置 Google Cloud Project ([console.cloud.google.com/projectcreate](https://console.cloud.google.com/projectcreate))
- 配置 Cloud Billing Account ([console.cloud.google.com/billing/create](https://console.cloud.google.com/billing/create))
- 将 Billing Account 链接到 Project ([console.cloud.google.com/billing/projects](https://console.cloud.google.com/billing/projects))
- 将您的 Domain 添加到 Cloudflare ([Add site to Cloudflare](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/#add-site-to-cloudflare) 和 [您的 Domain Registrar](https://www.youtube.com/watch?v=uqlo3lCqiy0))
- 注册 Cloudflare Zero Trust Free plan ([one.dash.cloudflare.com](https://one.dash.cloudflare.com/))

## 步骤 1：创建 Google Compute Engine VM 实例

- 点击 <button class="button button--solid-blue">Create Instance</button> ([VM Instances ↗](https://console.cloud.google.com/compute/instances))，启用 Compute Engine API (如果需要) 并配置：
- **Machine configuration**
    - Name: `n8n-free-tier` (或您偏好的 Name)
    - Region: `us-west1 (Oregon)` (或您偏好的 Region: *us-west1 (Oregon)*, *us-central1 (Iowa)*, *us-east1 (South Carolina)*)
    - Machine Type: `e2-micro`
- **Operating system and storage** (OS and storage)
    - 点击 <button class="button button--solid-blue">Change</button> 并配置：
        - Operating system: `Container Optimized OS`
        - Boot disk type: `Standard persistent disk`
        - Size (GB): `30`
    - 点击 <button class="button button--solid-blue">Select</button>
- **Data protection**
    - 选择 `No backups`
- **Networking**
    - 切换您的 Network interface (Network interfaces)
        - 从 Premium 切换到 `Standard` (Network Service Tier)
- **Observability**
    - 取消勾选 `Install Ops Agent for Monitoring and Logging`
- 点击 <button class="button button--solid-blue">Create</button>

## 步骤 2：创建 Cloudflare Tunnel

- 点击 <button class="button button--solid-blue">Create a tunnel</button> ([Zero Trust ↗](https://one.dash.cloudflare.com/) > Networks > Tunnels)
- 点击 <button class="button button--outline-blue">Select Cloudflared</button>
- 命名您的 Tunnel (例如，`n8n-gcp-tunnel`)
- 点击 <button class="button button--solid-blue">Save tunnel</button>
- 点击 <button class="button button--solid-blue">Docker</button>
- 从显示的 Command 中复制 Token value (即 --token 后面的字母数字 String)。
- 为您的 Tunnel 添加 Public hostname
    - **Subdomain:** (例如，`n8n`)
    - **Domain:** (例如，`mydomain.com`)
    - **Type:** `HTTP`
    - **URL:** `n8n:5678`
- 点击 <button class="button button--solid-blue">Complete Setup</button>

## 步骤 3：使用 Docker 安装 n8n

- 通过 `SSH` 连接到您的 Google Cloud VM ([Google Cloud console ↗](console.cloud.google.com) > Compute Engine > VM instances > Connect)
- 创建目录
    ```bash
    mkdir ~/n8n-cloudflare
    cd ~/n8n-cloudflare
    ```
- 创建 .env 文件
    ```bash
    nano .env
    ```
- 粘贴下方 .env 文件部分的内容。请记住替换 `CLOUDFLARE_TUNNEL_TOKEN` 和 `N8N_HOST` 的占位符值。保存并退出 (Ctrl+X, Y, Enter)。
    ```bash
    # .env file

    # Cloudflare Tunnel Token - REPLACE THIS WITH YOUR ACTUAL TOKEN
    # You got this when setting up the tunnel in the Cloudflare Zero Trust dashboard.
    # IMPORTANT: Keep this file secure and do NOT commit it to version control!
    CLOUDFLARE_TUNNEL_TOKEN="eyJhIj..."

    # N8N Configuration
    # The primary domain where n8n will be accessible.
    N8N_HOST=n8n.mydomain.com

    # Timezone for n8n. Find your timezone string here: [https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
    GENERIC_TIMEZONE=America/New_York

    # N8N Basic Authentication (HIGHLY RECOMMENDED FOR SECURITY)
    # Set to 'true' to enable basic authentication.
    N8N_BASIC_AUTH_ACTIVE=true
    ```
- 设置 .env 的权限
    ```bash
    chmod 600 .env
    ```
- 创建 docker-compose.yml 文件
    ```bash
    nano docker-compose.yml
    ```
- 粘贴上方 docker-compose.yml 部分的内容。保存并退出
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
- 启动服务
    ```bash
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v "$(pwd):/app" -w /app docker/compose:latest up -d
    ```
现在，当您在浏览器中访问 https://n8n.mydomain.com.com 时，Cloudflare 将通过其网络，通过隧道，直接将流量路由到您的 n8n 容器，而无需暴露您的 VM 的 IP 地址。

## 清理

即将推出：关于如何轻松彻底地从 Google Cloud 中删除 N8N 设置和相关资源的详细说明。

## 优点

- **零入站流量：** 无需勾选防火墙规则下的 `允许 HTTPS 流量`，可防止您的 VM 及其宝贵数据直接暴露在公共互联网上。
- **轻松 HTTPS：** 无需担心安装和管理 SSL 证书。Cloudflare 为您自动化所有 HTTPS 终止。无需 Nginx 或 Certbot 等工具。
- **动态 IP 自由：** 忘掉静态 IP 地址或复杂的 DNS 管理。您的 VM 与 Cloudflare 的边缘网络建立持久的、仅出站的连接，允许您的 VM 的 IP 地址更改，同时您的 N8N 域名保持可访问。您可以随时自信地重启或停止您的实例，而无需担心 IP 地址的变化。
- **容器优化性能：** 您的 N8N 实例运行在 Google 的容器优化操作系统 (Container-Optimized OS) 上，这是一种从头开始构建的专用操作系统，用于高效可靠地运行容器化应用程序，超越了 Debian 等常用替代方案。
- **零成本企业级：** 利用 Google Cloud 免费套餐和 Cloudflare 的企业级网络，以零成本部署强大、生产就绪的 N8N 实例。

## 局限性

- 每月在特定美国区域 (俄勒冈：us-west1、爱荷华：us-central1、南卡罗来纳：us-east1) 提供一个非抢占式 e2-micro VM 实例
- 30 GB 标准持久性磁盘存储
- 每月 200 GB 免费网络出站流量
- 无日志记录
- 无监控
- 无快照计划

## 故障排除和常见问题

即将推出：您可能遇到的常见问题的解决方案，以及有用的故障排除技巧，可帮助您快速恢复正常。

## 进一步增强

- 添加单独的 webhook 主机名
- 使用外部数据库 (例如，PostgreSQL)

## 资源

- [Free Tier features \| Google Cloud](https://cloud.google.com/free/docs/free-cloud-features#free-tier)
- [Container-Optimized OS \| Google Cloud](https://youtu.be/jh0fPT-AWwM?t=211)
- [Container-Optimized OS Overview \| Google Cloud](https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits)
- [Create a tunnel (dashboard) \| Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)
- [Hosting n8n on Google Cloud \| n8n Docs](https://docs.n8n.io/hosting/installation/server-setups/google-cloud/)
- [Self-Hosting n8n with Docker & Cloudflare Tunnel \| n8n Community](https://community.n8n.io/t/securely-self-hosting-n8n-with-docker-cloudflare-tunnel-the-arguably-less-painful-way/93801)
- [GCP Network Service Tiers (Cloud Performance Atlas)](https://www.youtube.com/watch?v=wsdgWGE-mwE)
- [Cloudflare origin CA](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)
