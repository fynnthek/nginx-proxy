# nginx-proxy

Nginx reverse proxy to make docker containers accessible from the outside via http/https. This was built for my personal use case but might be useful for some people. :)

## Installation
Clone this repository into any folder and copy data/nginx/conf.d/default.conf.example into data/nginx/conf.d/default.conf. Modify and copy the example virtual host for as many domains as you need.

### Example using https
Place your certifcates at data/ssl/my.domain.name/
```nginx
{
server {
    listen 80; # Remove this line if you want to configure redirecting http to https.
    listen 443 ssl http2;
    server_name my.domain.name;

    # Path for SSL config/key/certificate
    ssl_certificate /etc/ssl/certs/nginx/my.domain.name/fullchain.pem;
    ssl_certificate_key /etc/ssl/certs/nginx/my.domain.name/fullchain.key;
    include /etc/nginx/includes/ssl.conf;

    location / {
        include /etc/nginx/includes/proxy.conf;
        proxy_pass http://host.docker.internal:8080;
    }

    access_log off;
    error_log /var/log/nginx/error.log error;
}
```

### Example without using https
```nginx
{
server {
    listen 80;
    server_name my.domain.name;

    location / {
        include /etc/nginx/includes/proxy.conf;
        proxy_pass http://host.docker.internal:8080;
    }

    access_log off;
    error_log /var/log/nginx/error.log error;
}
```

Spin up the container via docker-compose. It will restart automatically.
```
docker-compose up -d
```

## Redirect http to https
Copy data/nginx/conf.d/redirect.conf.example into data/nginx/conf.d/redirect.conf and add the domain names you want to redirect to https as server_name. Remove ```listen 80;``` from the respective virtual host in data/nginx/conf.d/default.conf.
```nginx
{
server {
    listen 80;
    server_name my.domain.name;
    server_name my.seconddomain.name;

    return 302 https://$host$request_uri;
}
```

## Credits
Build around [this guide](https://phoenixnap.com/kb/docker-nginx-reverse-proxy) but modified to use local files instead of placing them inside the container itself. SSL configuration created with [Mozilla SSL Configuration generator](https://ssl-config.mozilla.org).
