# ServerConfiguration NGINX
We will compile the NGINX ourself.
At the time of writing, the last version is : nginx-1.11.2.tar.gz

## Download the source tarball at :http://nginx.org/download/
## Avoid information disclosure
To avoid information disclosure about the web server, we will to spoof Microsoft-IIS/8.5  as our web server,
edit the file **ngx_http_header_filter_module.c** in the **src/http/** folder:

```python
vi +48 src/http/ngx_http_header_filter_module.c
```
Now operate these changes: 
```python
~~static char ngx_http_server_string[] = "Server: nginx" CRLF;~~
static char ngx_http_server_string[] = "Server: Microsoft-IIS/8.5 " CRLF;
~~static char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;~~
static char ngx_http_server_full_string[] = "Server: Microsoft-IIS/8.5 " NGINX_VER CRLF;

```

## Compile with this options to some modules that are will not be used
```python
 ./configure --without-http_autoindex_module --without-http_ssi_module
  make 
  make install
```
Install PCRE in case of problem
```python
 install libpcre3 libpcre3-dev
```
So this is the result:
```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```
## Test if installation is OKAY
```python
 sudo /usr/local/nginx/sbin/nginx -Vt 
```
You will get this result:
```
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
nginx version: nginx/1.11.2
built by gcc 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) 
configure arguments: --without-http_autoindex_module --without-http_ssi_module
```
## To start Nginx, use :
```python
/usr/local/nginx/sbin/nginx
```
# Web server configuration options

The **/usr/local/nginx/conf/nginx.conf** configuration file must contain these security options 

```python
http {
        ##
        # Basic Settings
        # NEVER BROADCAST THE NGINX VERSION NUMBER IN ERROR PAGES AND SERVER HEADER
        server_tokens off;
        # WE WILL NOT DISCLOSE INFORMATION THE REQUEST IS A UNAUTHORIZED 401 , FORBIDDEN 403 OR NOT FOUND 404 
        error_page 401 403 404 /404.html;
        #WE DON'T WANT TO BE IFRAMED AND MITIGATE CLICKJACKING ATTACK
        add_header X-Frame-Options SAMEORIGIN;
        #PROTECT SOME BROWSER FROM SNIFFING THE CONTENT TYPE
        add_header X-Content-Type-Options nosniff;
        # MITIGATE XSS ATTACKS..
        add_header X-XSS-Protection "1; mode=block";
        
        #SERVER OPTIONS
        server {
          listen 443 ssl default deferred;
          server_name .ourwebsite.com;
          
          # OUR CERTS AND KEYS
          ssl_certificate /etc/nginx/ssl/star_ourwebsite_com.crt;
          ssl_certificate_key /etc/nginx/ssl/star_ourwebsite_com.key;
        
          # ENABLE SESSION RESUMPTION TO IMPROVE HTTPS PERFORMANCE
          ssl_session_cache shared:SSL:10m;
          ssl_session_timeout 5m;
          keepalive_timeout 70;
        
          # DIFFIE-HELLMAN PARAMETER FOR DHE CIPHERSUITES, RECOMMENDED 4096 BITS
          ssl_dhparam /etc/nginx/ssl/dhparam.pem;
        
          # ENABLES SERVER-SIDE PROTECTION FROM BEAST ATTACKS
          ssl_prefer_server_ciphers on;
          # disable SSLv3(enabled by default since nginx 0.8.19) since it's less secure then TLS
          ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
          # CIPHERS CHOSEN FOR FORWARD SECRECY AND COMPATIBILITY
          ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:ECDHE-RSA-AES128-GCM-SHA256:AES256+EECDH:DHE-RSA-AES128-GCM-SHA256:AES256+EDH:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
        
          # ENABLE OCSP STAPLING 
          resolver 8.8.8.8;
          ssl_stapling on;
          ssl_trusted_certificate /etc/nginx/ssl/star_ourwebsite_com.crt;
        
          # CONFIG TO ENABLE HSTS (HTTP Strict Transport Security)
          add_header Strict-Transport-Security "max-age=31536000; includeSubdomains;";
        
          # ... the rest of our configuration
        }
        
        # REDIRECT ALL HTTP TRAFFIC TO HTTPS
        server {
          listen 80;
          server_name .ourwebsite.com;
          return 301 https://$host$request_uri;
        }




...
```

Then usr/local/nginx/sbin/nginx -s reload
# SSL configuration


```python

```















