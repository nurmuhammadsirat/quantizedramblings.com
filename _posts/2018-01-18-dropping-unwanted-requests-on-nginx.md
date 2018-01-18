---
title: Dropping Unwanted Requests On Nginx
asset_path: /assets/dropping-unwanted-requests-on-nginx
updated: 2018-01-18 21:38
---

When I host an application behind an Nginx server, I usually don't want any other requests hitting it. This is especially if I know exactly the specific endpoints that is exposed. An example Nginx config file for a Rails application deployed on a Unix socket would look something like this:

````
upstream app_server {
  server unix:///var/www/my-rails-app/tmp/sockets/my-rails-app.sock;
}

server {
  listen 80;
  server_name my-rails-app.com;

  root /var/www/my-rails-app/public;
  access_log /var/www/my-rails-app/log/nginx.access.log;
  error_log /var/www/my-rails-app/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://app_server;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
````

Now because I'm not serving a generic application, but something with specific endpoints that I know, I can replace the ````location /```` block with this:

````
location /endpoint-1 {
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_redirect off;

  proxy_pass http://app_server;
}

location /endpoint-2 {
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_redirect off;

  proxy_pass http://app_server;
}
````

And then specifically reject all other requests by placing this right at the top:

````
location / {
  deny all;
}
````

Now, my config file looks like this:

````
upstream app_server {
  server unix:///var/www/my-rails-app/tmp/sockets/my-rails-app.sock;
}

server {
  listen 80;
  server_name my-rails-app.com;

  root /var/www/my-rails-app/public;
  access_log /var/www/my-rails-app/log/nginx.access.log;
  error_log /var/www/my-rails-app/log/nginx.error.log info;

  location / {
    deny all;
  }

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location /endpoint-1 {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://app_server;
  }

  location /endpoint-2 {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://app_server;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
````


However, if I were to do a curl to an endpoint that is not passed through to the Rails application, this is what is returned:

````
$ curl http://my-rails-app.com
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.10.3</center>
</body>
</html>
$
````

There's still minimal HTML! What if I don't even want that? What if I want the Nginx web server to drop the request and not return anything? Not even a 403?

Turns out, Nginx has a custom built-in method to handle that. We simply do a [return 444](http://nginx.org/en/docs/http/request_processing.html), which is Nginx's special sauce that closes the connection immediately.

So now, my config file would look like this:

````
upstream app_server {
  server unix:///var/www/my-rails-app/tmp/sockets/my-rails-app.sock;
}

server {
  listen 80;
  server_name my-rails-app.com;

  root /var/www/my-rails-app/public;
  access_log /var/www/my-rails-app/log/nginx.access.log;
  error_log /var/www/my-rails-app/log/nginx.error.log info;

  location / {
    return 444;
  }

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location /endpoint-1 {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://app_server;
  }

  location /endpoint-2 {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://app_server;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
````

And if I try to send a request, the connection is immediately discarded.

````
curl http://my-rails-app.com/
curl: (52) Empty reply from server
````
