#Nginx: 
---------
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


========================================================= Main Part From Here =====================================================
#/etc/nginx/nginx.conf => 
----------------------------

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
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
307 is the http code (means => redirect other location ); ; 

#if baseUrl/crops ; not found then redirect /fruits 

location /crops{
    return 307 /fruits
}

 location /fruits {
            root /User/ashraful/mysite # baseUrl/fruits => also show the index.html file . 
        }
