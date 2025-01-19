# Detailed Nginx Configuration Guide with Line-by-Line Comments

## Basic Laravel Site Configuration

```nginx
# Define a new server block (virtual host)
server {
    # Listen on port 80 (HTTP)
    listen 80;
    
    # Define your domain name
    server_name your-domain.com;
    
    # Set the root directory to Laravel's public folder
    root /var/www/your-laravel-app/public;

    # Prevent clickjacking attacks by restricting frame embedding to same origin
    add_header X-Frame-Options "SAMEORIGIN";
    
    # Prevent MIME-type sniffing exploits
    add_header X-Content-Type-Options "nosniff";

    # Set default index file (Laravel's index.php)
    index index.php;

    # Set character encoding
    charset utf-8;

    # Main location block for handling Laravel's front controller pattern
    location / {
        # Try to serve the requested URI, then the URI with a trailing slash,
        # finally fall back to the Laravel front controller (index.php)
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Disable logging for favicon and robots.txt
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    # Set error page for 404 to be handled by Laravel
    error_page 404 /index.php;

    # PHP-FPM configuration block
    location ~ \.php$ {
        # Pass PHP requests to PHP-FPM via Unix socket
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        
        # Set the script filename for PHP-FPM
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        
        # Include standard FastCGI parameters
        include fastcgi_params;
    }

    # Deny access to hidden files (starting with .)
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## SSL/TLS Configuration

```nginx
server {
    # Listen on port 443 with SSL and HTTP/2 enabled
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL certificate paths (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # Specify supported SSL/TLS protocols
    # Only allow TLS 1.2 and 1.3 (more secure versions)
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Define allowed cipher suites (encryption algorithms)
    # These are modern, secure cipher suites supporting Perfect Forward Secrecy
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    
    # Let clients choose the cipher (recommended for modern setups)
    ssl_prefer_server_ciphers off;
    
    # SSL session settings for better performance
    ssl_session_timeout 1d;          # Hold SSL session info for 1 day
    ssl_session_cache shared:SSL:50m; # Shared SSL cache of 50mb
    ssl_session_tickets off;         # Disable TLS session tickets
    
    # Enable HTTP Strict Transport Security
    # Forces browsers to use HTTPS for future visits
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

## Caching Configuration

```nginx
# Define FastCGI cache location and settings
fastcgi_cache_path /tmp/nginx_cache  # Cache directory
    levels=1:2                       # Two-level directory hierarchy for better performance
    keys_zone=laravel_cache:100m     # Allocate 100MB for cache keys
    max_size=10g                     # Maximum cache size
    inactive=60m                     # Remove items not accessed for 60 minutes
    use_temp_path=off;              # Write directly to cache dir

server {
    location ~ \.php$ {
        # Enable FastCGI caching
        fastcgi_cache laravel_cache;
        
        # Cache successful responses for 60 minutes
        fastcgi_cache_valid 200 60m;
        
        # Use stale cache if backend is down or errors occur
        fastcgi_cache_use_stale error timeout updating http_500 http_503;
        
        # Cache after first request
        fastcgi_cache_min_uses 1;
        
        # Prevent multiple requests from updating cache simultaneously
        fastcgi_cache_lock on;
        
        # Allow cache bypass with Cache-Control header
        fastcgi_cache_bypass $http_cache_control;
        fastcgi_no_cache $http_cache_control;
    }

    # Browser caching for static assets
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|woff2)$ {
        # Cache in browser for 1 year
        expires 1y;
        
        # Allow public caching, prevent transformation
        add_header Cache-Control "public, no-transform";
    }
}
```

## Load Balancing Configuration

```nginx
# Define upstream servers (backend application servers)
upstream laravel_backends {
    # Simple round-robin load balancing
    server backend1.example.com:80;  # First backend server
    server backend2.example.com:80;  # Second backend server
    server backend3.example.com:80;  # Third backend server
}

# Advanced load balancing configuration
upstream laravel_backends_advanced {
    # Use IP hash for session persistence
    ip_hash;  
    
    # Server with weight 3 (receives more traffic)
    server backend1.example.com:80 weight=3;
    
    # Server with weight 2
    server backend2.example.com:80 weight=2;
    
    # Backup server (used when others are down)
    server backend3.example.com:80 backup;
}

server {
    location / {
        # Forward requests to the backend servers
        proxy_pass http://laravel_backends;
        
        # Set proper headers for proxying
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## Security Configurations

```nginx
# Rate limiting zone definition
# Limit based on client IP, allocate 10MB for tracking
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

# DDoS protection settings
# Track concurrent connections per IP
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

server {
    # Security headers
    # Prevent iframe embedding except from same origin
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Enable browser's XSS protection
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Prevent MIME type sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # Control referrer information
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    
    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Rate limiting for login endpoint
    location /login {
        # Apply rate limiting with burst of 5 requests
        limit_req zone=login burst=5 nodelay;
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Global connection limiting
    limit_conn conn_limit_per_ip 10;  # Maximum 10 concurrent connections per IP
}
```

## Performance Optimization

```nginx
# Gzip compression settings
gzip on;                     # Enable gzip compression
gzip_vary on;               # Send Vary: Accept-Encoding header
gzip_proxied any;           # Compress for all proxied requests
gzip_comp_level 6;          # Compression level (1-9, higher = more compression but more CPU)
gzip_types                  # MIME types to compress
    text/plain
    text/css
    text/xml
    application/json
    application/javascript
    application/xml+rss
    application/atom+xml
    image/svg+xml;

# Buffer size configurations
client_body_buffer_size 10K;            # Buffer size for POST submissions
client_header_buffer_size 1k;           # Buffer size for most headers
client_max_body_size 8m;                # Maximum allowed request body size
large_client_header_buffers 2 1k;       # Buffer size for large headers

# Worker process settings
worker_processes auto;                  # Auto-detect number of CPU cores
worker_rlimit_nofile 65535;            # Maximum number of open files per worker

events {
    multi_accept on;                    # Accept multiple connections per worker
    worker_connections 65535;           # Maximum connections per worker
    use epoll;                         # Use efficient epoll event processing
}
```

## Logging Configuration

```nginx
# Define custom log format with timing information
log_format timing '$remote_addr - $remote_user [$time_local] '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent" '
                  '$request_time $upstream_response_time $pipe';

# Main access log configuration
access_log /var/log/nginx/access.log timing buffer=512k;  # Use custom format with buffer
error_log /var/log/nginx/error.log warn;                 # Error log with warning level

# Server status monitoring
location /nginx_status {
    stub_status on;           # Enable basic status information
    access_log off;           # Disable access logging for status page
    allow 127.0.0.1;          # Only allow localhost access
    deny all;                 # Deny all other access
}
```

## Maintenance Mode Configuration

```nginx
# Check for maintenance file
if (-f $document_root/maintenance.html) {
    return 503;  # Return Service Unavailable status
}

# Define maintenance page
error_page 503 @maintenance;
location @maintenance {
    rewrite ^(.*)$ /maintenance.html break;  # Show maintenance page
}
```

Remember that these configurations should be adjusted based on your specific needs and server capabilities. Always test configurations in a staging environment before applying them to production.
