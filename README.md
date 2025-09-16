# OpenSIPS Installation Guide

OpenSIPS is a free and open-source SIP Server. Multi-functional SIP server

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5060 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5060 (default opensips port)
- **Dependencies**:
  - opensips-mysql-module
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install opensips
sudo dnf install -y opensips opensips-mysql-module

# Enable and start service
sudo systemctl enable --now opensips

# Configure firewall
sudo firewall-cmd --permanent --add-service=opensips
sudo firewall-cmd --reload

# Verify installation
opensips --version || systemctl status opensips
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install opensips
sudo apt install -y opensips opensips-mysql-module

# Enable and start service
sudo systemctl enable --now opensips

# Configure firewall
sudo ufw allow 5060

# Verify installation
opensips --version || systemctl status opensips
```

### Arch Linux

```bash
# Install opensips
sudo pacman -S opensips

# Enable and start service
sudo systemctl enable --now opensips

# Verify installation
opensips --version || systemctl status opensips
```

### Alpine Linux

```bash
# Install opensips
apk add --no-cache opensips

# Enable and start service
rc-update add opensips default
rc-service opensips start

# Verify installation
opensips --version || rc-service opensips status
```

### openSUSE/SLES

```bash
# Install opensips
sudo zypper install -y opensips opensips-mysql-module

# Enable and start service
sudo systemctl enable --now opensips

# Configure firewall
sudo firewall-cmd --permanent --add-service=opensips
sudo firewall-cmd --reload

# Verify installation
opensips --version || systemctl status opensips
```

### macOS

```bash
# Using Homebrew
brew install opensips

# Start service
brew services start opensips

# Verify installation
opensips --version
```

### FreeBSD

```bash
# Using pkg
pkg install opensips

# Enable in rc.conf
echo 'opensips_enable="YES"' >> /etc/rc.conf

# Start service
service opensips start

# Verify installation
opensips --version || service opensips status
```

### Windows

```powershell
# Using Chocolatey
choco install opensips

# Or using Scoop
scoop install opensips

# Verify installation
opensips --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/opensips

# Set up basic configuration
sudo tee /etc/opensips/opensips.conf << 'EOF'
# OpenSIPS Configuration
children=4
EOF

# Test configuration
sudo opensips -t || sudo opensips configtest

# Reload service
sudo systemctl reload opensips
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R opensips:opensips /etc/opensips
sudo chmod 750 /etc/opensips

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable opensips

# Start service
sudo systemctl start opensips

# Stop service
sudo systemctl stop opensips

# Restart service
sudo systemctl restart opensips

# Reload configuration
sudo systemctl reload opensips

# Check status
sudo systemctl status opensips

# View logs
sudo journalctl -u opensips -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add opensips default

# Start service
rc-service opensips start

# Stop service
rc-service opensips stop

# Restart service
rc-service opensips restart

# Check status
rc-service opensips status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'opensips_enable="YES"' >> /etc/rc.conf

# Start service
service opensips start

# Stop service
service opensips stop

# Restart service
service opensips restart

# Check status
service opensips status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start opensips
brew services stop opensips
brew services restart opensips

# Check status
brew services list | grep opensips
```

### Windows Service Manager

```powershell
# Start service
net start opensips

# Stop service
net stop opensips

# Using PowerShell
Start-Service opensips
Stop-Service opensips
Restart-Service opensips

# Check status
Get-Service opensips
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/opensips/opensips.conf << 'EOF'
children=4
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart opensips
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream opensips_backend {
    server 127.0.0.1:5060;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name opensips.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name opensips.example.com;

    ssl_certificate /etc/ssl/certs/opensips.example.com.crt;
    ssl_certificate_key /etc/ssl/private/opensips.example.com.key;

    location / {
        proxy_pass http://opensips_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName opensips.example.com
    Redirect permanent / https://opensips.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName opensips.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/opensips.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/opensips.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5060/
    ProxyPassReverse / http://127.0.0.1:5060/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5060/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend opensips_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/opensips.pem
    redirect scheme https if !{ ssl_fc }
    default_backend opensips_backend

backend opensips_backend
    balance roundrobin
    option httpchk GET /health
    server opensips1 127.0.0.1:5060 check
    server opensips2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R opensips:opensips /etc/opensips
sudo chmod 750 /etc/opensips

# Configure firewall
sudo firewall-cmd --permanent --add-service=opensips
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/opensips.conf << 'EOF'
[opensips]
enabled = true
port = 5060
filter = opensips
logpath = /var/log/opensips/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/opensips.key \
    -out /etc/ssl/certs/opensips.crt

# Configure SSL in opensips
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE opensips_db;
CREATE USER opensips_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE opensips_db TO opensips_user;
EOF

# Configure opensips to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE opensips_db;
CREATE USER 'opensips_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON opensips_db.* TO 'opensips_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# OpenSIPS specific tuning
children=4
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
opensips soft nofile 65535
opensips hard nofile 65535
opensips soft nproc 32768
opensips hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'opensips'
    static_configs:
      - targets: ['localhost:5060']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet opensips; then
    echo "OpenSIPS is running"
    exit 0
else
    echo "OpenSIPS is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/opensips << 'EOF'
/var/log/opensips/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 opensips opensips
    postrotate
        systemctl reload opensips > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# OpenSIPS backup script
BACKUP_DIR="/backup/opensips"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop opensips

# Backup configuration
tar -czf "$BACKUP_DIR/opensips-config-$DATE.tar.gz" /etc/opensips

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/opensips-data-$DATE.tar.gz" /var/lib/opensips

# Start service
systemctl start opensips

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop opensips

# Restore configuration
sudo tar -xzf /backup/opensips/opensips-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/opensips/opensips-data-*.tar.gz -C /

# Set permissions
sudo chown -R opensips:opensips /etc/opensips
sudo chown -R opensips:opensips /var/lib/opensips

# Start service
sudo systemctl start opensips
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u opensips -n 100
sudo tail -f /var/log/opensips/*.log

# Check configuration
sudo opensips -t || sudo opensips configtest

# Check permissions
ls -la /etc/opensips
ls -la /var/lib/opensips
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5060
sudo netstat -tlnp | grep 5060

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5060
nc -zv localhost 5060
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep opensips)
htop -p $(pgrep opensips)

# Check connections
ss -ant | grep :5060 | wc -l

# Monitor I/O
iotop -p $(pgrep opensips)
```

### Debug Mode

```bash
# Run in debug mode
sudo opensips -d
# or
sudo opensips debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  opensips:
    image: opensips:latest
    container_name: opensips
    ports:
      - "5060:5060"
    volumes:
      - ./config:/etc/opensips
      - ./data:/var/lib/opensips
    environment:
      - opensips_CONFIG=/etc/opensips/opensips.conf
    restart: unless-stopped
    networks:
      - opensips_net

networks:
  opensips_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: opensips
spec:
  replicas: 3
  selector:
    matchLabels:
      app: opensips
  template:
    metadata:
      labels:
        app: opensips
    spec:
      containers:
      - name: opensips
        image: opensips:latest
        ports:
        - containerPort: 5060
        volumeMounts:
        - name: config
          mountPath: /etc/opensips
      volumes:
      - name: config
        configMap:
          name: opensips-config
---
apiVersion: v1
kind: Service
metadata:
  name: opensips
spec:
  selector:
    app: opensips
  ports:
  - port: 5060
    targetPort: 5060
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure OpenSIPS
  hosts: all
  become: yes
  tasks:
    - name: Install opensips
      package:
        name: opensips
        state: present
    
    - name: Configure opensips
      template:
        src: opensips.conf.j2
        dest: /etc/opensips/opensips.conf
        owner: opensips
        group: opensips
        mode: '0640'
      notify: restart opensips
    
    - name: Start and enable opensips
      systemd:
        name: opensips
        state: started
        enabled: yes
  
  handlers:
    - name: restart opensips
      systemd:
        name: opensips
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update opensips

# Debian/Ubuntu
sudo apt update && sudo apt upgrade opensips

# Arch Linux
sudo pacman -Syu opensips

# Alpine Linux
apk update && apk upgrade opensips

# openSUSE
sudo zypper update opensips

# FreeBSD
pkg update && pkg upgrade opensips

# Always backup before updates
tar -czf /backup/opensips-pre-update-$(date +%Y%m%d).tar.gz /etc/opensips

# Restart after updates
sudo systemctl restart opensips
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/opensips -name "*.log" -mtime +30 -delete

# Verify integrity
sudo opensips --verify || sudo opensips check

# Update databases (if applicable)
sudo opensips-update-db

# Optimize performance
sudo opensips-optimize

# Check for security updates
sudo opensips --security-check
```

## Additional Resources

- Official Documentation: https://docs.opensips.org/
- GitHub Repository: https://github.com/opensips/opensips
- Community Forum: https://forum.opensips.org/
- Wiki: https://wiki.opensips.org/
- Comparison vs Kamailio, Asterisk, FreeSWITCH, repro: https://docs.opensips.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
