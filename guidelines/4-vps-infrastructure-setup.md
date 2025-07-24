# 4. Hướng Dẫn Setup Infrastructure VPS/VDS cho BSC Validators

## Tổng Quan

Hướng dẫn chi tiết cách setup và configure VPS/VDS infrastructure để vận hành BSC validators và fullnode một cách chuyên nghiệp.

## 🏗️ Kiến Trúc Infrastructure Khuyến Nghị

### Minimal Setup (4-6 Validators)

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   VPS-VAL-01    │    │   VPS-VAL-02    │    │   VPS-VAL-03    │
│   Validator 1   │    │   Validator 2   │    │   Validator 3   │
│   2vCPU/8GB     │    │   2vCPU/8GB     │    │   2vCPU/8GB     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   VPS-FULL-01   │
                    │   Full Node     │
                    │   4vCPU/16GB    │
                    │   Public RPC    │
                    └─────────────────┘
```

### Production Setup (10+ Validators)

```
                     ┌─── Load Balancer ────┐
                     │                     │
        ┌─────────────────┐    ┌─────────────────┐
        │   VPS-FULL-01   │    │   VPS-FULL-02   │
        │   Full Node     │    │   Full Node     │
        │   8vCPU/32GB    │    │   8vCPU/32GB    │
        └─────────────────┘    └─────────────────┘
                     │                     │
    ┌────────────────┼─────────────────────┼────────────────┐
    │                │                     │                │
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ VPS-VAL-01  │ │ VPS-VAL-02  │ │ VPS-VAL-03  │ │ VPS-VAL-N   │
│ Validator 1 │ │ Validator 2 │ │ Validator 3 │ │ Validator N │
│ 4vCPU/16GB  │ │ 4vCPU/16GB  │ │ 4vCPU/16GB  │ │ 4vCPU/16GB  │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
```

## 💰 Yêu Cầu Hardware và Pricing

### Validator Node Requirements

```bash
# Minimum (Development/Testnet)
CPU: 2 vCPUs
RAM: 8GB
Storage: 500GB SSD
Network: 1 Gbps
OS: Ubuntu 20.04+ LTS

# Recommended (Production)
CPU: 4 vCPUs
RAM: 16GB
Storage: 1TB NVMe SSD
Network: 1 Gbps with low latency
OS: Ubuntu 22.04 LTS

# Premium (High-traffic Production)
CPU: 8 vCPUs
RAM: 32GB
Storage: 2TB NVMe SSD
Network: 10 Gbps with premium bandwidth
OS: Ubuntu 22.04 LTS
```

### Full Node Requirements

```bash
# Basic (Internal use)
CPU: 4 vCPUs
RAM: 16GB
Storage: 1TB SSD
Network: 1 Gbps

# Public RPC (External users)
CPU: 8 vCPUs
RAM: 32GB
Storage: 2TB NVMe SSD
Network: 10 Gbps

# High-volume RPC (DApp/Exchange)
CPU: 16 vCPUs
RAM: 64GB
Storage: 4TB NVMe SSD
Network: 10 Gbps with CDN
```

### Estimated Monthly Costs (USD)

```bash
# Cloud Providers (AWS/GCP/Azure)
Validator (Minimum): $50-80/month
Validator (Production): $150-250/month
Full Node (Public): $300-500/month

# Dedicated Providers (OVH/Hetzner/DigitalOcean)
Validator (Minimum): $25-40/month
Validator (Production): $80-120/month
Full Node (Public): $150-250/month

# Local Data Centers (varies by region)
Validator (Production): $60-100/month
Full Node (Public): $120-200/month
```

## 🌍 Provider Selection và Setup

### Recommended VPS Providers

#### Tier 1: Enterprise (High reliability)

```bash
# AWS EC2
- Pros: Global presence, enterprise support, scalability
- Cons: Higher cost, complex pricing
- Best for: Large scale production deployments

# Google Cloud Platform
- Pros: Fast networks, good pricing for sustained use
- Cons: Complex setup, billing complexity
- Best for: Tech-savvy teams, global deployment

# Microsoft Azure
- Pros: Enterprise integration, hybrid cloud options
- Cons: Windows-centric, complex networking
- Best for: Enterprise customers with existing MS infrastructure
```

#### Tier 2: Developer-Friendly (Good balance)

```bash
# DigitalOcean
- Pros: Simple pricing, good documentation, SSD standard
- Cons: Limited global presence, fewer enterprise features
- Best for: Small to medium teams, straightforward deployments

# Linode (Akamai)
- Pros: Excellent performance/price, good support
- Cons: Smaller network, fewer advanced services
- Best for: Performance-focused teams, cost-conscious deployments

# Vultr
- Pros: Global presence, competitive pricing, good performance
- Cons: Smaller provider, less enterprise support
- Best for: Global deployment needs, budget-conscious teams
```

#### Tier 3: Cost-Effective (Budget options)

```bash
# Hetzner
- Pros: Excellent price/performance, dedicated options
- Cons: Limited to Europe/US, German company (data residency)
- Best for: EU-based teams, cost optimization

# OVHcloud
- Pros: Very competitive pricing, European focus
- Cons: Support quality varies, complex interface
- Best for: European deployment, budget optimization

# Contabo
- Pros: Very low prices, decent performance
- Cons: Limited support, basic features
- Best for: Development/testing, extreme budget constraints
```

### Multi-Region Deployment Strategy

```bash
# Global Distribution Example
Region 1 (US-East): Validator-01, Validator-02
Region 2 (EU-West): Validator-03, Validator-04
Region 3 (Asia-Pacific): Validator-05, Validator-06
Full Nodes: One in each region for redundancy
```

## 🔧 VPS Initial Setup

### Bước 1: Server Provisioning

```bash
# DigitalOcean Example (using doctl)
# Install doctl first: snap install doctl

# Create SSH key
doctl compute ssh-key create bsc-validator --public-key-file ~/.ssh/id_rsa.pub

# Create validator droplets
for i in {1..4}; do
    doctl compute droplet create "bsc-validator-0$i" \
        --size s-4vcpu-8gb \
        --image ubuntu-22-04-x64 \
        --region nyc3 \
        --ssh-keys $(doctl compute ssh-key list --format ID --no-header) \
        --enable-monitoring \
        --enable-backups
done

# Create full node
doctl compute droplet create "bsc-fullnode-01" \
    --size s-8vcpu-16gb \
    --image ubuntu-22-04-x64 \
    --region nyc3 \
    --ssh-keys $(doctl compute ssh-key list --format ID --no-header) \
    --enable-monitoring \
    --enable-backups

# List created droplets
doctl compute droplet list
```

### Bước 2: Security Hardening

```bash
# Script chạy trên mỗi VPS sau khi tạo
#!/bin/bash

# Update system
apt update && apt upgrade -y

# Install essential packages
apt install -y curl wget git htop iotop nethogs ufw fail2ban

# Configure firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 30303   # BSC P2P
ufw allow 8545    # RPC (có thể restrict theo IP)
ufw allow 8546    # WebSocket
ufw --force enable

# Configure fail2ban
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF

systemctl restart fail2ban

# Disable root login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd

# Create BSC user
useradd -m -s /bin/bash bsc
usermod -aG sudo bsc

# Setup SSH key for bsc user
mkdir -p /home/bsc/.ssh
cp /root/.ssh/authorized_keys /home/bsc/.ssh/
chown -R bsc:bsc /home/bsc/.ssh
chmod 700 /home/bsc/.ssh
chmod 600 /home/bsc/.ssh/authorized_keys

echo "✅ Security hardening completed"
```

### Bước 3: Performance Optimization

```bash
# Script tối ưu performance
#!/bin/bash

# Configure kernel parameters for blockchain workload
cat >> /etc/sysctl.conf << 'EOF'
# Network optimization
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 65536 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_congestion_control = bbr

# File system optimization
fs.file-max = 1000000
fs.nr_open = 1000000

# Virtual memory optimization
vm.swappiness = 10
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
vm.vfs_cache_pressure = 50
EOF

sysctl -p

# Configure limits
cat >> /etc/security/limits.conf << 'EOF'
bsc soft nofile 1000000
bsc hard nofile 1000000
bsc soft nproc 1000000
bsc hard nproc 1000000
EOF

# Configure systemd limits
mkdir -p /etc/systemd/system.conf.d
cat > /etc/systemd/system.conf.d/limits.conf << 'EOF'
[Manager]
DefaultLimitNOFILE=1000000
DefaultLimitNPROC=1000000
EOF

systemctl daemon-reload

# SSD optimization (if applicable)
if lsblk | grep -q nvme; then
    echo "Detected NVMe SSD, applying optimizations..."
    echo 'ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"' > /etc/udev/rules.d/60-nvme-scheduler.rules
    echo 'ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"' >> /etc/udev/rules.d/60-nvme-scheduler.rules
fi

echo "✅ Performance optimization completed"
```

### Bước 4: Monitoring Setup

```bash
# Install Node Exporter for Prometheus monitoring
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.6.1.linux-amd64*

# Create systemd service
sudo tee /etc/systemd/system/node_exporter.service > /dev/null << 'EOF'
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=bsc
Group=bsc
Type=simple
ExecStart=/usr/local/bin/node_exporter --web.listen-address=:9100

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Allow monitoring port
sudo ufw allow 9100

echo "✅ Monitoring setup completed"
```

## 📊 Load Balancer Setup (cho Full Nodes)

### Nginx Load Balancer Configuration

```bash
# Trên VPS riêng biệt cho load balancer
apt install -y nginx

# Configure load balancer
cat > /etc/nginx/sites-available/bsc-rpc << 'EOF'
upstream bsc_rpc {
    least_conn;
    server fullnode-01.example.com:8545 max_fails=3 fail_timeout=30s;
    server fullnode-02.example.com:8545 max_fails=3 fail_timeout=30s;
    server fullnode-03.example.com:8545 max_fails=3 fail_timeout=30s backup;
}

upstream bsc_ws {
    least_conn;
    server fullnode-01.example.com:8546 max_fails=3 fail_timeout=30s;
    server fullnode-02.example.com:8546 max_fails=3 fail_timeout=30s;
    server fullnode-03.example.com:8546 max_fails=3 fail_timeout=30s backup;
}

server {
    listen 80;
    listen 443 ssl http2;
    server_name rpc.yournetwork.com;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/rpc.yournetwork.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/rpc.yournetwork.com/privkey.pem;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=rpc_limit:10m rate=10r/s;
    limit_req zone=rpc_limit burst=20 nodelay;

    # RPC endpoint
    location / {
        proxy_pass http://bsc_rpc;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # Headers
        add_header Access-Control-Allow-Origin "*";
        add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        add_header Access-Control-Allow-Headers "Content-Type";
    }

    # WebSocket endpoint
    location /ws {
        proxy_pass http://bsc_ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
EOF

# Enable site
ln -s /etc/nginx/sites-available/bsc-rpc /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

echo "✅ Load balancer configured"
```

### HAProxy Alternative

```bash
# Install HAProxy
apt install -y haproxy

# Configure HAProxy
cat > /etc/haproxy/haproxy.cfg << 'EOF'
global
    daemon
    maxconn 4096
    log stdout local0

defaults
    mode http
    timeout connect 5s
    timeout client 60s
    timeout server 60s
    option httplog

frontend bsc_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/bsc.pem
    redirect scheme https if !{ ssl_fc }

    # Rate limiting
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request reject if { sc_http_req_rate(0) gt 20 }

    default_backend bsc_backend

backend bsc_backend
    balance roundrobin
    option httpchk GET /
    http-check expect status 200

    server fullnode-01 fullnode-01.example.com:8545 check inter 5s
    server fullnode-02 fullnode-02.example.com:8545 check inter 5s
    server fullnode-03 fullnode-03.example.com:8545 check inter 5s backup

listen stats
    bind *:8404
    stats enable
    stats uri /
    stats refresh 30s
    stats admin if TRUE
EOF

systemctl restart haproxy
echo "✅ HAProxy configured"
```

## 🔒 SSL/TLS Configuration

### Let's Encrypt Setup

```bash
# Install Certbot
apt install -y certbot python3-certbot-nginx

# Get SSL certificate
certbot --nginx -d rpc.yournetwork.com -d ws.yournetwork.com

# Setup auto-renewal
cat > /etc/cron.d/certbot << 'EOF'
0 12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew --nginx
EOF

echo "✅ SSL/TLS configured"
```

## 🔍 Health Checks và Monitoring

### Health Check Script

```bash
# Tạo health check script
cat > /usr/local/bin/bsc-health-check.sh << 'EOF'
#!/bin/bash

# BSC Node Health Check Script
NODE_TYPE=$1  # validator or fullnode
RPC_URL="http://localhost:8545"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo "=== BSC Node Health Check ($NODE_TYPE) ==="
echo "Time: $(date)"

# Check service status
if [ "$NODE_TYPE" = "validator" ]; then
    SERVICE="bsc-validator"
else
    SERVICE="bsc-fullnode"
fi

if systemctl is-active --quiet $SERVICE; then
    echo -e "${GREEN}✅ Service Status: Active${NC}"
else
    echo -e "${RED}❌ Service Status: Inactive${NC}"
    exit 1
fi

# Check RPC connectivity
if curl -s -f $RPC_URL > /dev/null; then
    echo -e "${GREEN}✅ RPC Connectivity: OK${NC}"
else
    echo -e "${RED}❌ RPC Connectivity: Failed${NC}"
    exit 1
fi

# Check sync status
SYNC_STATUS=$(curl -s -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
    -H "Content-Type: application/json" $RPC_URL | jq -r .result)

if [ "$SYNC_STATUS" = "false" ]; then
    echo -e "${GREEN}✅ Sync Status: Synced${NC}"
else
    echo -e "${YELLOW}⏳ Sync Status: Syncing${NC}"
    echo "   Details: $SYNC_STATUS"
fi

# Check latest block
LATEST_BLOCK=$(curl -s -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
    -H "Content-Type: application/json" $RPC_URL | jq -r .result)
BLOCK_NUM=$((16#${LATEST_BLOCK#0x}))
echo "📊 Latest Block: $BLOCK_NUM"

# Check peer count
PEER_COUNT=$(curl -s -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' \
    -H "Content-Type: application/json" $RPC_URL | jq -r .result)
PEERS=$((16#${PEER_COUNT#0x}))
echo "🌐 Peer Count: $PEERS"

if [ $PEERS -lt 3 ]; then
    echo -e "${YELLOW}⚠️  Warning: Low peer count${NC}"
fi

# Check if validator is mining
if [ "$NODE_TYPE" = "validator" ]; then
    MINING=$(curl -s -X POST --data '{"jsonrpc":"2.0","method":"eth_mining","params":[],"id":1}' \
        -H "Content-Type: application/json" $RPC_URL | jq -r .result)
    if [ "$MINING" = "true" ]; then
        echo -e "${GREEN}⛏️  Mining: Active${NC}"
    else
        echo -e "${YELLOW}⚠️  Mining: Inactive${NC}"
    fi
fi

# Check disk usage
DISK_USAGE=$(df /opt/bsc/data | tail -1 | awk '{print $5}' | sed 's/%//')
echo "💾 Disk Usage: ${DISK_USAGE}%"

if [ $DISK_USAGE -gt 85 ]; then
    echo -e "${RED}❌ Warning: High disk usage${NC}"
elif [ $DISK_USAGE -gt 70 ]; then
    echo -e "${YELLOW}⚠️  Warning: Moderate disk usage${NC}"
fi

# Check memory usage
MEM_USAGE=$(free | grep Mem | awk '{printf "%.1f", $3/$2 * 100.0}')
echo "🧠 Memory Usage: ${MEM_USAGE}%"

echo "=== End Health Check ==="
EOF

chmod +x /usr/local/bin/bsc-health-check.sh

# Setup cron job
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/bsc-health-check.sh validator >> /var/log/bsc-health.log 2>&1") | crontab -

echo "✅ Health checks configured"
```

## 🔄 Backup và Disaster Recovery

### Automated Backup Script

```bash
cat > /usr/local/bin/bsc-backup.sh << 'EOF'
#!/bin/bash

# BSC Node Backup Script
BACKUP_DIR="/backup/bsc"
RETENTION_DAYS=7
S3_BUCKET="your-bsc-backups"  # Optional

# Create backup directory
mkdir -p $BACKUP_DIR

# Stop node for consistent backup
systemctl stop bsc-validator || systemctl stop bsc-fullnode

# Backup data directory
BACKUP_FILE="$BACKUP_DIR/bsc-data-$(date +%Y%m%d-%H%M%S).tar.gz"
tar -czf $BACKUP_FILE -C /opt/bsc data/

# Backup configuration
CONFIG_BACKUP="$BACKUP_DIR/bsc-config-$(date +%Y%m%d-%H%M%S).tar.gz"
tar -czf $CONFIG_BACKUP -C /opt/bsc config.toml genesis.json

# Start node
systemctl start bsc-validator || systemctl start bsc-fullnode

# Upload to S3 (optional)
if [ -n "$S3_BUCKET" ] && command -v aws >/dev/null; then
    aws s3 cp $BACKUP_FILE s3://$S3_BUCKET/
    aws s3 cp $CONFIG_BACKUP s3://$S3_BUCKET/
fi

# Cleanup old backups
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "✅ Backup completed: $BACKUP_FILE"
EOF

chmod +x /usr/local/bin/bsc-backup.sh

# Schedule daily backups
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/bsc-backup.sh >> /var/log/bsc-backup.log 2>&1") | crontab -
```

## 📈 Scaling Strategies

### Horizontal Scaling

```bash
# Add more validators
1. Provision new VPS
2. Run security hardening
3. Deploy BSC node
4. Add to static nodes list
5. Update monitoring

# Add more full nodes
1. Provision new VPS
2. Deploy full node
3. Add to load balancer
4. Update health checks
```

### Vertical Scaling

```bash
# Upgrade existing nodes
1. Create backup
2. Stop node
3. Resize VPS (CPU/RAM/Storage)
4. Start node
5. Verify performance
```

## 📝 Checklist Infrastructure Setup

### ✅ Pre-Deployment

- [ ] VPS providers selected và evaluated
- [ ] SSH keys generated và distributed
- [ ] Network architecture planned
- [ ] Security policies defined

### ✅ VPS Setup

- [ ] Servers provisioned
- [ ] Security hardening completed
- [ ] Performance optimization applied
- [ ] Monitoring agents installed

### ✅ Load Balancing (nếu cần)

- [ ] Load balancer configured
- [ ] SSL certificates installed
- [ ] Rate limiting configured
- [ ] Health checks implemented

### ✅ Monitoring

- [ ] Health check scripts deployed
- [ ] Prometheus/Grafana setup (optional)
- [ ] Alert mechanisms configured
- [ ] Log aggregation setup

### ✅ Backup & Recovery

- [ ] Backup scripts configured
- [ ] Recovery procedures tested
- [ ] Off-site storage configured
- [ ] Documentation updated

🎯 **Infrastructure sẵn sàng cho BSC deployment!**
