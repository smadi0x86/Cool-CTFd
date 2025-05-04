# Basic Deployment Guide

This guide will help you deploy CTFd and CTFd-Whale on a single VPS.

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/CoolCTFd.git
cd CoolCTFd
```

### 2. Initialize Docker Swarm

Initialize a Docker swarm and label your node:

```bash
docker swarm init
docker node update --label-add "name=linux-1" $(docker node ls -q)
```

### 3. Configure frps

Create the necessary directories and configuration files:

```bash
mkdir -p conf/frp
```

Create `conf/frp/frps.ini` with the following content:

```ini
[common]
bind_port = 7987
vhost_http_port = 8001
token = your_token
subdomain_host = your-domain.com
```

### 4. Configure frpc

Create `conf/frp/frpc.ini` with the following content:

```ini
[common]
token = your_token
server_addr = frps
server_port = 7987
admin_addr = 0.0.0.0
admin_port = 7400
```

### 5. Start Containers

```bash
docker-compose up -d
```

### 6. Configure CTFd-Whale Plugin

Access the Whale Configuration page at `/plugins/ctfd-whale/admin/settings` and configure the following settings:

#### Docker Settings
- **API URL**: `unix:///var/run/docker.sock`
- **Credentials**: Leave empty for local Docker
- **Swarm Nodes**: `linux-1` or `windows-1`
- **Use SSL**: Unchecked

#### Standalone Containers
- **Auto Connect Network**: `cool-ctfd_frp_containers`
- **Dns Setting**: `1.1.1.1`

#### Grouped Containers
- **Auto Connect Containers**: Leave empty
- **Multi-Container Network Subnet**: `174.1.0.0/16`
- **Multi-Container Network Subnet New Prefix**: `24`

#### Router Settings
- **Router type**: `frp`
- **API URL**: `http://frpc:7400`
- **Http Domain Suffix**: `your-domain.com`
- **External Http Port**: `8001`
- **Direct IP Address**: `your-domain.com`
- **Direct Minimum Port**: `10000`
- **Direct Maximum Port**: `10100`
- **Frpc config template**:

```ini
[common]
token = your_token
server_addr = frps
server_port = 7987
admin_addr = 0.0.0.0
admin_port = 7400
```

### 7. Configure nginx (Optional)

If you are using CTFd 2.5.0+, you can utilize the included nginx.

Add the following server block to `./conf/nginx/http.conf`:

```conf
server {
  listen 80;
  server_name *.your-domain.com;
  location / {
    proxy_pass http://frps:8001;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $server_name;
  }
}
```

### Verifying frp Configuration
To verify that frpc/frps is set up correctly, check the logs:

```bash
docker-compose logs frpc frps
```
You should see logs indicating successful connection between frpc and frps.

## Security Considerations
- Do not set bind_addr of the frpc to `0.0.0.0` if you are following this guide. This may enable contestants to override frpc configurations.
- If you are annoyed by the complicated configuration, and you just want to set bind_addr = 0.0.0.0, remember to enable Basic Auth included in frpc, and set API URL accordingly, for example, `http://username:password@frpc:7400`
