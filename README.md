# Modoboa Installation Guide

Modoboa is a free and open-source Mail Server Stack. Mail hosting made simple

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
  - Network: 443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default modoboa port)
- **Dependencies**:
  - python3, postgresql, nginx
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

# Install modoboa
sudo dnf install -y modoboa python3, postgresql, nginx

# Enable and start service
sudo systemctl enable --now modoboa

# Configure firewall
sudo firewall-cmd --permanent --add-service=modoboa
sudo firewall-cmd --reload

# Verify installation
modoboa --version || systemctl status modoboa
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install modoboa
sudo apt install -y modoboa python3, postgresql, nginx

# Enable and start service
sudo systemctl enable --now modoboa

# Configure firewall
sudo ufw allow 443

# Verify installation
modoboa --version || systemctl status modoboa
```

### Arch Linux

```bash
# Install modoboa
sudo pacman -S modoboa

# Enable and start service
sudo systemctl enable --now modoboa

# Verify installation
modoboa --version || systemctl status modoboa
```

### Alpine Linux

```bash
# Install modoboa
apk add --no-cache modoboa

# Enable and start service
rc-update add modoboa default
rc-service modoboa start

# Verify installation
modoboa --version || rc-service modoboa status
```

### openSUSE/SLES

```bash
# Install modoboa
sudo zypper install -y modoboa python3, postgresql, nginx

# Enable and start service
sudo systemctl enable --now modoboa

# Configure firewall
sudo firewall-cmd --permanent --add-service=modoboa
sudo firewall-cmd --reload

# Verify installation
modoboa --version || systemctl status modoboa
```

### macOS

```bash
# Using Homebrew
brew install modoboa

# Start service
brew services start modoboa

# Verify installation
modoboa --version
```

### FreeBSD

```bash
# Using pkg
pkg install modoboa

# Enable in rc.conf
echo 'modoboa_enable="YES"' >> /etc/rc.conf

# Start service
service modoboa start

# Verify installation
modoboa --version || service modoboa status
```

### Windows

```powershell
# Using Chocolatey
choco install modoboa

# Or using Scoop
scoop install modoboa

# Verify installation
modoboa --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/modoboa

# Set up basic configuration
sudo tee /etc/modoboa/modoboa.conf << 'EOF'
# Modoboa Configuration
processes = 4
EOF

# Test configuration
sudo modoboa -t || sudo modoboa configtest

# Reload service
sudo systemctl reload modoboa
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R modoboa:modoboa /etc/modoboa
sudo chmod 750 /etc/modoboa

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable modoboa

# Start service
sudo systemctl start modoboa

# Stop service
sudo systemctl stop modoboa

# Restart service
sudo systemctl restart modoboa

# Reload configuration
sudo systemctl reload modoboa

# Check status
sudo systemctl status modoboa

# View logs
sudo journalctl -u modoboa -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add modoboa default

# Start service
rc-service modoboa start

# Stop service
rc-service modoboa stop

# Restart service
rc-service modoboa restart

# Check status
rc-service modoboa status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'modoboa_enable="YES"' >> /etc/rc.conf

# Start service
service modoboa start

# Stop service
service modoboa stop

# Restart service
service modoboa restart

# Check status
service modoboa status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start modoboa
brew services stop modoboa
brew services restart modoboa

# Check status
brew services list | grep modoboa
```

### Windows Service Manager

```powershell
# Start service
net start modoboa

# Stop service
net stop modoboa

# Using PowerShell
Start-Service modoboa
Stop-Service modoboa
Restart-Service modoboa

# Check status
Get-Service modoboa
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/modoboa/modoboa.conf << 'EOF'
processes = 4
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart modoboa
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
upstream modoboa_backend {
    server 127.0.0.1:443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name modoboa.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name modoboa.example.com;

    ssl_certificate /etc/ssl/certs/modoboa.example.com.crt;
    ssl_certificate_key /etc/ssl/private/modoboa.example.com.key;

    location / {
        proxy_pass http://modoboa_backend;
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
    ServerName modoboa.example.com
    Redirect permanent / https://modoboa.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName modoboa.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/modoboa.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/modoboa.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend modoboa_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/modoboa.pem
    redirect scheme https if !{ ssl_fc }
    default_backend modoboa_backend

backend modoboa_backend
    balance roundrobin
    option httpchk GET /health
    server modoboa1 127.0.0.1:443 check
    server modoboa2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R modoboa:modoboa /etc/modoboa
sudo chmod 750 /etc/modoboa

# Configure firewall
sudo firewall-cmd --permanent --add-service=modoboa
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/modoboa.conf << 'EOF'
[modoboa]
enabled = true
port = 443
filter = modoboa
logpath = /var/log/modoboa/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/modoboa.key \
    -out /etc/ssl/certs/modoboa.crt

# Configure SSL in modoboa
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE modoboa_db;
CREATE USER modoboa_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE modoboa_db TO modoboa_user;
EOF

# Configure modoboa to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE modoboa_db;
CREATE USER 'modoboa_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON modoboa_db.* TO 'modoboa_user'@'localhost';
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

# Modoboa specific tuning
processes = 4
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
modoboa soft nofile 65535
modoboa hard nofile 65535
modoboa soft nproc 32768
modoboa hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'modoboa'
    static_configs:
      - targets: ['localhost:443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet modoboa; then
    echo "Modoboa is running"
    exit 0
else
    echo "Modoboa is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/modoboa << 'EOF'
/var/log/modoboa/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 modoboa modoboa
    postrotate
        systemctl reload modoboa > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Modoboa backup script
BACKUP_DIR="/backup/modoboa"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop modoboa

# Backup configuration
tar -czf "$BACKUP_DIR/modoboa-config-$DATE.tar.gz" /etc/modoboa

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/modoboa-data-$DATE.tar.gz" /var/lib/modoboa

# Start service
systemctl start modoboa

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop modoboa

# Restore configuration
sudo tar -xzf /backup/modoboa/modoboa-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/modoboa/modoboa-data-*.tar.gz -C /

# Set permissions
sudo chown -R modoboa:modoboa /etc/modoboa
sudo chown -R modoboa:modoboa /var/lib/modoboa

# Start service
sudo systemctl start modoboa
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u modoboa -n 100
sudo tail -f /var/log/modoboa/*.log

# Check configuration
sudo modoboa -t || sudo modoboa configtest

# Check permissions
ls -la /etc/modoboa
ls -la /var/lib/modoboa
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443
sudo netstat -tlnp | grep 443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443
nc -zv localhost 443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep uwsgi)
htop -p $(pgrep uwsgi)

# Check connections
ss -ant | grep :443 | wc -l

# Monitor I/O
iotop -p $(pgrep uwsgi)
```

### Debug Mode

```bash
# Run in debug mode
sudo modoboa -d
# or
sudo modoboa debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  modoboa:
    image: modoboa:latest
    container_name: modoboa
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/modoboa
      - ./data:/var/lib/modoboa
    environment:
      - modoboa_CONFIG=/etc/modoboa/modoboa.conf
    restart: unless-stopped
    networks:
      - modoboa_net

networks:
  modoboa_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: modoboa
spec:
  replicas: 3
  selector:
    matchLabels:
      app: modoboa
  template:
    metadata:
      labels:
        app: modoboa
    spec:
      containers:
      - name: modoboa
        image: modoboa:latest
        ports:
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /etc/modoboa
      volumes:
      - name: config
        configMap:
          name: modoboa-config
---
apiVersion: v1
kind: Service
metadata:
  name: modoboa
spec:
  selector:
    app: modoboa
  ports:
  - port: 443
    targetPort: 443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Modoboa
  hosts: all
  become: yes
  tasks:
    - name: Install modoboa
      package:
        name: modoboa
        state: present
    
    - name: Configure modoboa
      template:
        src: modoboa.conf.j2
        dest: /etc/modoboa/modoboa.conf
        owner: modoboa
        group: modoboa
        mode: '0640'
      notify: restart modoboa
    
    - name: Start and enable modoboa
      systemd:
        name: modoboa
        state: started
        enabled: yes
  
  handlers:
    - name: restart modoboa
      systemd:
        name: modoboa
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update modoboa

# Debian/Ubuntu
sudo apt update && sudo apt upgrade modoboa

# Arch Linux
sudo pacman -Syu modoboa

# Alpine Linux
apk update && apk upgrade modoboa

# openSUSE
sudo zypper update modoboa

# FreeBSD
pkg update && pkg upgrade modoboa

# Always backup before updates
tar -czf /backup/modoboa-pre-update-$(date +%Y%m%d).tar.gz /etc/modoboa

# Restart after updates
sudo systemctl restart modoboa
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/modoboa -name "*.log" -mtime +30 -delete

# Verify integrity
sudo modoboa --verify || sudo modoboa check

# Update databases (if applicable)
sudo modoboa-update-db

# Optimize performance
sudo modoboa-optimize

# Check for security updates
sudo modoboa --security-check
```

## Additional Resources

- Official Documentation: https://docs.modoboa.org/
- GitHub Repository: https://github.com/modoboa/modoboa
- Community Forum: https://forum.modoboa.org/
- Wiki: https://wiki.modoboa.org/
- Comparison vs mailcow, Mailu, iRedMail, Postal: https://docs.modoboa.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
