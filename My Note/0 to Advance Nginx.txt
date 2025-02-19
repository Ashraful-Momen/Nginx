#Nginx: 
-------
#Defination : 
-------------
is a web server that serve the web content to the browser . That also support SSL/ TLS(encryption ), Network load balancing (support 1 http request to send multiple web server of replicas or instance) . 

#install the Nginx with the docker command : 
--------------------------------------------
#download the img: 
>>> docker pull nginx 

#run the Nginx to the 81 port: 
-------------------------------
┌──(ashraful㉿kali)-[~]
└─$ docker run -itd --rm --name nginx_container  -p 81:80 nginx

#access the ssh of the Nginx : 
------------------------------
┌──(ashraful㉿kali)-[~]
└─$ docker exe -it container_id_of_Nginx bash

#config file of the Nginx : 
---------------------------
>>> cd /etc/nginx/nginx.conf

#refer the other files : 
-----------------------------
 include /etc/nginx/conf.d/*.conf;  #here is the main file that set up the root file and etc file configuration .

#index.html file => 
--------------------
root@4c0aa00778f2:/usr/share/nginx/html# ls
50x.html  index.html


**** After change anything in nginx.config , we have use this command for the relaod : 
>>> nginx -s reload 


# for docker container reload after configuration change : 
docker exec -it c367dc nginx -t  # Test config (should return successful)
docker exec -it c367dc nginx -s reload  # Reload Nginx

#check the container log: 
docker logs -f c367dc  # Show all logs

#for only the access log : 
docker exec -it c367dc tail -f /var/log/nginx/access.log

#for the error log : 
docker exec -it c367dc tail -f /var/log/nginx/error.log


========================================================= Main Part From Here =====================================================
#/etc/nginx/nginx.conf => 
----------------------------

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;  // 1 worker_processes can handle 1024 http Request connection . We Can customize the value 
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;  #here is the main file that set up the root file and etc file configuration .
}


================================================== explain the command ====================================================
#directives is known as key value pairs | ex : worker_processor 1 

#context : the block of the code | events { worker_connections  1024; }


#--------------------------------------------
#command =>  'worker_processor 1'; 


explain => 
-----------

worker_processes 1;

    This setting limits Nginx to one worker process.
    It is suitable for low-traffic sites or environments with a single CPU core.
    This can be a bottleneck if the server has multiple CPU cores because Nginx won't utilize the available resources efficiently.
    
2. worker_processes auto;

worker_processes auto;

    Nginx automatically sets the number of worker processes equal to the number of CPU cores available.
    This is the recommended setting for multi-core servers, as it maximizes performance.
    You can check the number of CPU cores with:

grep -c ^processor /proc/cpuinfo

If your server has 4 CPU cores, auto will assign worker_processes 4;.

3. Custom Values (worker_processes N;)

worker_processes 4;

    This explicitly sets 4 worker processes, useful when you want to fine-tune performance.
    Generally, the ideal number of worker processes is equal to the number of CPU cores.

Which One Should You Use?

    For single-core systems → Use worker_processes 1;
    For multi-core systems → Use worker_processes auto; (best practice)
    For fine-tuning → Use a custom value based on your workload and CPU availability.
#--------------------------------------

explain the context => http{}: 
------------------------

#serve the http block/context: 
http {

    #this block configure the server 
    server {
    
        #it listen on port : 8080
        listen 8080; 
        
        #show the file to the browser
        root /Users/ashraful/mysite;  [inside the folder I put index.html file]
    
    }
}

**** After change anything in nginx.config , we have use this command for the relaod : 
>>> nginx -s reload 

#-----------------------------------------

explain the MIME type block/context => type {include file_path}
-------------------------------------

http{
    server{
        
        #add the css/js/photo etc file here . 
        type{
         text/css css; 
         text/html html; 
         
        }
        
        listen 81;
        root /User/ashraful/mysite
    }
}

#all the file in mime.types => here include all kinds of type 

http{
    server{
        
        #add the css/js/photo etc file here .all the file types declear in the file location .  
        include mime.types 
        
        listen 81;
        root /User/ashraful/mysite
    }
}

# context  => location {//refer the url | baseUrl/location_url}: 
=> location redirect/other_file
=> alias redirect/other_location
---------------------------

http{
    server{
        
        #add the css/js/photo etc file here .all the file types declear in the file location .  
        include mime.types 
        
        listen 81;
        root /User/ashraful/mysite
        
        location /fruits {
            root /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
        
        #when hit this => http://baseUrl/flowers/ ; also show the /friuts folder location .  
        location /flower {
            alias /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
        
        
    }
}

#----------------------------------
=> when we hit the => http:baseUrl/fruits ; automatically browser search the index.html file in the folder =>  root /User/ashraful/mysite # baseUrl/fruits ; 
if not found then generate the error 403; forbidden cause index.html file not found . 

#to solve the problem use directive inside the location context => try_files file_location.html

#


http{
    server{
        
        #add the css/js/photo etc file here .all the file types declear in the file location .  
        include mime.types 
        
        listen 81;
        root /User/ashraful/mysite
        
        location /fruits {
            root /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
        
        #when hit this => http://baseUrl/flowers/ ; also show the /friuts folder location .  
        location /flower {
            alias /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
        
        
        #main part for the => try_file value: ____________________________________________ 
        location /vegitable {
            root /User/ashraful/mysite/vagitable;       # if index.html file not found by browser then show the 403 error ; 
            
            try_file vegitable.html                     # if index.html file not found then search vagitable.html
            
            try_file vagitable.html /index.html =404    # if vagitable.html file not found then search index.html(this is the error page ) , if still not found(index.html errorPage) then generate 404 error. 
        }
        
        
    }
}

#-------------------------------------------------
=> Use Regular Expression for redirect the file location => 
=> hit baseUrl/count/6 ; that redirect file => 

#baseUrl/count/9 : show the vagitable.html if exit , if not exit then show the index.html errorPage , if error page not found then show the 404 error . 
 location ~*/count/[0-9] {
            root /User/ashraful/mysite/vagitable;       # if index.html file not found by browser then show the 403 error ; 
            
            try_file vegitable.html                     # if index.html file not found then search vagitable.html
            
            try_file vagitable.html /index.html =404    # if vagitable.html file not found then search index.html(this is the error page ) , if still not found(index.html errorPage) then generate 404 error. 
        }

#-------------------------------------------------
307 is the http code (means => redirect other location );

#if baseUrl/crops ; not found then redirect /fruits 

location /crops{
    return 307 /fruits
}

 location /fruits {
            root /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
        
#--------------------------------------------------
# baseUrl/newUrlName/$value that redirect to another url or location /redirect_location_name{}

=> rewrite ^/number/(\w+)  /count/$1

=> rewrite baseUrl/number/take_value_from_url /redirectLocation/$pass_the_value

code : 
---------
http{
server1{

rewrite ^/number/(\w+)  /count/$1

 location ~*/count/[0-9] {
            root /User/ashraful/mysite/vagitable;       # if index.html file not found by browser then show the 403 error ; 
            
            try_file vegitable.html                     # if index.html file not found then search vagitable.html
            
            try_file vagitable.html /index.html =404    # if vagitable.html file not found then search index.html(this is the error page ) , if still not found(index.html errorPage) then generate 404 error. 
        }

}

server2{

}

}
#-------------------------------------------------------

#Request distributed to another server : 
----------------------------------------

http {
    upstream backend {
        # Define your upstream servers
        server 192.168.1.101;
        server 192.168.1.102;
        server 192.168.1.103;

        # Enable dynamic RPS distribution
        least_conn;  # Use least connections algorithm as a base
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;

            # Custom logic for dynamic RPS distribution
            # This can be implemented using Lua scripting or other methods
            # Below is a placeholder for custom logic
            # You can use Lua to dynamically adjust the weights of the upstream servers
            # based on the current RPS or other metrics.

            # Example Lua script (requires ngx_http_lua_module)
            # content_by_lua_block {
            #     local upstream = require "ngx.upstream"
            #     local peers = upstream.get_primary_peers("backend")
            #     for _, peer in ipairs(peers) do
            #         -- Adjust peer weight based on RPS or other metrics
            #         peer.weight = calculate_dynamic_weight(peer)
            #     end
            # }
        }
    }
}


#===================================================================== Header ======================================================================== 

location / {
    # Pass client information to the backend
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port $server_port;
    
    # Forward client browser and session information
    proxy_set_header User-Agent $http_user_agent;
    proxy_set_header Cookie $http_cookie;
    proxy_set_header Accept-Language $http_accept_language;
    proxy_set_header Referer $http_referer;
    
    # Forward client's authorization data
    proxy_set_header Authorization $http_authorization;
    
    # Custom headers (optional)
    proxy_set_header X-Custom-Header "MyAppSpecificValue";
}


Pass client information:

Host: Original hostname
X-Real-IP: Client's real IP  // important. 
X-Forwarded-For: Client IP chain
X-Forwarded-Proto: Original protocol (http/https)
X-Forwarded-Port: Original port


Forward browser and session information:

User-Agent: Browser identification
Cookie: Session cookies
Accept-Language: Client language preferences
Referer: Page that linked to this resource


Forward authorization data:

Authorization: Authentication credentials


Custom headers (optional example):

X-Custom-Header: "MyAppSpecificValue"

#----------------------------------------------
=========================================================== SSL Certificate Generate ======================================================
#OpenSSL command with notes explaining each part:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```

**Command breakdown:**

1. `openssl req` - Initiates a certificate request generation process
   * This starts the OpenSSL certificate creation utility

2. `-x509` - Tells OpenSSL to output a certificate in X.509 standard format
   * X.509 is the standard certificate format used for SSL/TLS

3. `-nodes` - Tells OpenSSL not to encrypt the private key with a passphrase
   * Makes automation easier but less secure (No DES Encryption)

4. `-days 365` - Specifies validity period of the certificate (1 year)
   * Self-signed certificates typically use shorter periods

5. `-newkey rsa:2048` - Creates a 2048-bit RSA key pair
   * RSA is the most common public-key encryption algorithm
   * 2048 bits provides good security while maintaining performance

6. `-keyout nginx-selfsigned.key` - Specifies output file for the private key
   * Private key must be kept secret
   * Used to decrypt data sent to the server

7. `-out nginx-selfsigned.crt` - Specifies output file for the certificate
   * Contains the public key
   * Shared with clients to establish encrypted connections

**Important notes:**
- The private key must be kept secure and never shared
- The certificate (.crt) contains the public key that clients use for encryption
- Self-signed certificates will trigger browser warnings
- For production, use certificates signed by a trusted Certificate Authority

============================================== How to set the open ssl certificate ===============================
#in localhost the defualt http port is 80 but when use the https with the ssl certificate then use 443. 

Here's the complete nginx configuration for HTTPS setup with detailed explanations:

```nginx
http {
    upstream nodejs_cluster {
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }

    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /Users/nana/nginx-certs/nginx-selfsigned.crt;
        ssl_certificate_key /Users/nana/nginx-certs/nginx-selfsigned.key;

        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

    server {
        listen 8080;
        server_name localhost;

        location / {
            return 301 https://$host$request_uri;
        }
    }
}
```

**Line-by-line explanation:**

1. `http { ... }` - Main HTTP context that encapsulates all HTTP-related configurations

2. `upstream nodejs_cluster { ... }` - Defines a group of servers for load balancing
3. `server 127.0.0.1:3002;` - First Node.js backend server running on localhost port 3002
4. `server 127.0.0.1:3003;` - Second Node.js backend server running on localhost port 3003

5. `server { ... }` - First server block for HTTPS traffic
6. `listen 443 ssl;` - Configures the server to listen on port 443 (standard HTTPS port) with SSL enabled
7. `server_name localhost;` - Sets the server name to respond to

8. `ssl_certificate /Users/nana/nginx-certs/nginx-selfsigned.crt;` - Path to SSL certificate file
9. `ssl_certificate_key /Users/nana/nginx-certs/nginx-selfsigned.key;` - Path to SSL private key file

10. `location / { ... }` - Configuration for handling requests to the root URL
11. `proxy_pass http://nodejs_cluster;` - Forwards requests to the upstream Node.js servers
12. `proxy_set_header Host $host;` - Passes the original host header to backend servers
13. `proxy_set_header X-Real-IP $remote_addr;` - Passes client's real IP to backends

14. `server { ... }` - Second server block for HTTP traffic redirection
15. `listen 8080;` - Configures this server to listen on port 8080 (HTTP)
16. `server_name localhost;` - Sets the server name to respond to

17. `location / { ... }` - Configuration for handling HTTP requests
18. `return 301 https://$host$request_uri;` - Permanent redirect (301) from HTTP to HTTPS
    - This redirects all HTTP traffic to HTTPS using the same hostname and request path

**Key aspects of this configuration:**
1. HTTPS server listens on port 443 with SSL enabled
2. Self-signed certificate is used for SSL/TLS encryption
3. Load balancing across multiple Node.js instances
4. HTTP to HTTPS redirection for security
5. Original client information preserved in requests to backends

#============================================= How to get the real rps ======================================
#Get the active client rps 

server {
    listen 80;
    
    location /nginx_status {
        stub_status; //this code help to get the real client rps 
        allow 127.0.0.1;  # Change this to your monitoring server if needed
        deny all;
    }
}


#Advance code for get : https / http RPS => 
---------------------------------------------
http {
    # Standard configuration
    log_format main_ext '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for" '
                        'rt=$request_time uct="$upstream_connect_time" '
                        'uht="$upstream_header_time" urt="$upstream_response_time"';
    
    # Enable stub_status module
    server {
        listen 8080;
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            deny all;
        }
    }
}
