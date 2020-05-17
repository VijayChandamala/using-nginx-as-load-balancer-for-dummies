# Using nginx as load balancer for dummies

Nginx can be used as reverse proxy and a load balancer.

Here's a simple example of setting up nginx using the nginx docker image.

This will run a temporary nginx container and copy the default conf file from the container to your current working directory

```
docker run --name tmp-nginx-container -d nginx

docker cp tmp-nginx-container:/etc/nginx/nginx.conf .

```

Once you have the default nginx conf, you can update it as per your requirement.

### nginx.conf
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
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

    #include /etc/nginx/conf.d/*.conf;

upstream apps { # simple upstream there are so many options like weightage and timeout etc..
        server 10.0.2.143:5000;
        server 10.0.2.82:5000;
        server 10.0.2.59:5000;
              }

server { # simple reverse-proxy
    listen       80;
    server_name  _;
    location / {
      proxy_pass      http://apps;
    }
  }
}

```
Once after updating the nginx.conf file

You can remove the temp nginx container and run actual nginx for reverse proxy and load balancing.

```
docker rm -f tmp-nginx-container

docker run --name my-nginx -p 80:80 -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx

```

This is a very simple version of it.
