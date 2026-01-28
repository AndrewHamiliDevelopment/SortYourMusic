# Deployment Guide

## Prerequisites

1. **Docker & Docker Compose** installed on your Ubuntu server
2. **Spotify Developer Credentials**:
   - Go to https://developer.spotify.com/dashboard
   - Create a new app
   - Note your Client ID
   - Add your redirect URI to the app settings (e.g., `http://your-server-ip:8080/` or `https://yourdomain.com/`)

## Setup Steps

### 1. Configure Spotify Credentials

Edit `web/config.js` and update:
```javascript
var SPOTIFY_CLIENT_ID = 'YOUR_CLIENT_ID_HERE';
var SPOTIFY_REDIRECT_URI = 'http://your-server-ip:8080/'; // or your domain
```

### 2. Build and Run with Docker Compose

```bash
# Build the image
docker-compose build

# Start the container
docker-compose up -d

# View logs
docker-compose logs -f
```

The application will be available at `http://your-server-ip:8080`

### 3. Stop the Application

```bash
docker-compose down
```

## Production Deployment with HTTPS

For production, you should use HTTPS. Here's a recommended setup with Nginx reverse proxy:

### 1. Install Nginx on Host

```bash
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx
```

### 2. Configure Nginx

Create `/etc/nginx/sites-available/sortyourmusic`:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:8080;
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

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/sortyourmusic /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 3. Set Up SSL with Let's Encrypt

```bash
sudo certbot --nginx -d yourdomain.com
```

### 4. Update Spotify Redirect URI

Update `web/config.js`:
```javascript
var SPOTIFY_REDIRECT_URI = 'https://yourdomain.com/';
```

Rebuild and restart:
```bash
docker-compose up -d --build
```

## Updating the Application

When you make changes:

```bash
# Rebuild and restart
docker-compose up -d --build

# Or if you only changed config.js (it's mounted as a volume):
docker-compose restart
```

## Troubleshooting

### Check container logs
```bash
docker-compose logs -f
```

### Check if container is running
```bash
docker-compose ps
```

### Access container shell
```bash
docker-compose exec sortyourmusic sh
```

### Port already in use
If port 8080 is in use, edit `docker-compose.yml` and change the port mapping:
```yaml
ports:
  - "8081:80"  # Use 8081 instead
```

## Environment Variables

You can customize the deployment by modifying `docker-compose.yml`:

- **Port**: Change `"8080:80"` to use a different host port
- **Timezone**: Modify `TZ` environment variable
- **Network**: Customize network settings if integrating with other services
