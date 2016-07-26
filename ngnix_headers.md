# ServerConfiguration NGINX
We will compile the NGINX ourself.
At the time of writing, the last version is : nginx-1.11.2.tar.gz

* Download the source tarball at :http://nginx.org/download/
* Avoid information disclosure
To avoid information disclosure about the web server, we will to spoof Microsoft-IIS/8.5  as our web server,
edit the file **ngx_http_header_filter_module.c** in the **src/http/** folder:

```python
static char ngx_http_server_string[] = "Server: nginx" CRLF;
static char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
```
by these: 

```python
static char ngx_http_server_string[] = "Server: Microsoft-IIS/8.5 ) " CRLF;
static char ngx_http_server_full_string[] = "Server: Microsoft-IIS/8.5 " NGINX_VER CRLF;
```
* Compile with this options to some modules that are will not be used
```python
 ./configure --without-http_autoindex_module --without-http_ssi_module
  make 
  make install
```


