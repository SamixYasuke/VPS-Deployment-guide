```markdown
# 🚀 VPS API - Deployment Guide

## 📦 Initial Server Setup

### Connect to EC2 Instance
```bash
cd $HOME/.ssh
ssh -i my-video-captioning-dev-key.pem ubuntu@13.48.28.126
```

### System Updates
```bash
sudo apt update
sudo apt upgrade -y
```

### Useful System Commands
| Command | Description |
|---------|-------------|
| `whoami` | Check current user context |
| `hostname -I` | Display network interfaces |
| `lsblk` | List attached storage |
| `df -h` | Check filesystem usage |
| `mkdir [folder_name]` | Create a new folder |
| `touch [file_name]` | Create a new file |
| `nano [file_name]` | Edit a file |
| `CTRL + O` | Save file in nano |
| `CTRL + X` | Exit nano editor |

---

## 🗄️ Redis Server Installation

### Install Redis
```bash
# Update package list and install Redis
sudo apt-get update
sudo apt-get install redis-server -y
```

### Configure Redis
```bash
# Enable Redis to start on boot
sudo systemctl enable redis-server

# Start Redis
sudo service redis-server start

# Check status
sudo service redis-server status

# Test connection
redis-cli ping
# Should return: PONG
```

### Redis Management Commands
```bash
# Stop Redis
sudo service redis-server stop

# Restart Redis
sudo service redis-server restart

# View Redis logs
sudo tail -f /var/log/redis/redis-server.log
```

---

## ⚙️ PM2 - Node.js Process Manager

### Starting Processes
```bash
# Start a process with custom name
pm2 start index.js --name [process_name]

# Example
pm2 start dist/index.js --name video-backend
```

### Process Management
| Command | Description |
|---------|-------------|
| `pm2 list` | List all PM2 processes |
| `pm2 logs [process_name]` | View logs for specific process |
| `pm2 logs` | View all logs |
| `pm2 restart [process_name]` | Restart a process |
| `pm2 stop [process_name]` | Stop a process |
| `pm2 delete [process_name]` | Delete a process |

### Auto-start on Server Reboot
```bash
# Generate startup script
pm2 startup

# This will output a command similar to:
# sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v18.16.0/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Run the generated command
sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v18.16.0/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu

# Save current PM2 processes
pm2 save
```

---

## 🌐 Nginx - Reverse Proxy Configuration

### DNS Configuration
Add an A record in your domain's DNS settings:

| Name | Type | Value / IP |
|------|------|------------|
| api  | A    | `<your EC2 public IPv4>` |

**Test DNS propagation:**
```bash
ping api.yourdomain.com
```

### Install Nginx
```bash
sudo apt update
sudo apt install nginx -y

# Check Nginx status
sudo systemctl status nginx
```

### Configure Nginx as Reverse Proxy

1. **Create configuration file:**
```bash
sudo nano /etc/nginx/sites-available/api.yourdomain.com
```

2. **Paste the configuration:**
```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;  # Node.js app port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. **Enable the site:**
```bash
sudo ln -s /etc/nginx/sites-available/api.yourdomain.com /etc/nginx/sites-enabled/
```

4. **Test and reload:**
```bash
# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

5. **Verify:**
Visit `http://api.yourdomain.com` - you should see your API response.

---

## 🔒 SSL Certificate with Certbot

### Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Issue SSL Certificate
```bash
sudo certbot --nginx -d api.yourdomain.com
```

Follow the interactive prompts:
- Enter email for renewal notifications
- Agree to terms of service
- Choose whether to redirect HTTP to HTTPS

### Test Auto-Renewal
```bash
sudo certbot renew --dry-run
```

### Verify HTTPS
Visit `https://api.yourdomain.com` - you should see a secure connection with the padlock icon.

---

## ✅ Post-Deployment Checklist

- [ ] Redis is running (`redis-cli ping` returns PONG)
- [ ] PM2 processes are running (`pm2 list`)
- [ ] Nginx configuration is valid (`sudo nginx -t`)
- [ ] Domain resolves to EC2 IP (`ping api.yourdomain.com`)
- [ ] HTTP site is accessible (`http://api.yourdomain.com`)
- [ ] HTTPS site is accessible (`https://api.yourdomain.com`)
- [ ] SSL auto-renewal is configured (`sudo certbot renew --dry-run`)

## 🔧 Useful Troubleshooting Commands

```bash
# Check if ports are listening
sudo netstat -tlnp

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check application logs
pm2 logs video-backend

# Check Redis connection
redis-cli ping

# Test Nginx configuration
sudo nginx -t

# Restart services
sudo systemctl restart nginx
pm2 restart video-backend
sudo service redis-server restart
```

---

## 📝 Environment Variables (.env)

Make sure to set these environment variables on your server:
```env
NODE_ENV=production
PORT=3000
MONGODB_PROD_URI=your_mongodb_connection_string
SESSION_SECRET=your_secure_session_secret
REDIS_HOST=localhost
REDIS_PORT=6379
FRONTEND_URL=https://your-frontend-domain.com
```

---

Your API should now be live at `https://api.yourdomain.com`! 🎉
```
