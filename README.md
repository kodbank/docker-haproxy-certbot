# HAProxy with Certbot
Based on [nmarus/docker-haproxy-certbot](https://github.com/nmarus/docker-haproxy-certbot)
For detailed info see origin repo.

## Changes
* Moved to Alpine Linux
* Updated HAProxy to 1.7.9
* Added check script ```(docker exec -it container_name haproxy-check -) < path_to_your_config```

## Usage

### 1. Create container
```
docker run -d \
  --restart=always \
  --name haproxy-certbot \
  --net=bridge \
  --cap-add=NET_ADMIN \
  -p 80:80 \
  -p 443:443 \
  -v /docker/haproxy/config:/config \
  -v /docker/haproxy/letsencrypt:/etc/letsencrypt \
  -v /docker/haproxy/certs.d:/usr/local/etc/haproxy/certs.d \
  dock1100/haproxy-certbot
```

### 2. Check config is valid
```
docker exec -it container_name haproxy-check
```
or
```
docker run -it --rm \
  -v /docker/haproxy/config/haproxy.cfg:/haproxy.cfg:ro \
  -v /docker/haproxy/letsencrypt:/etc/letsencrypt \
  -v /docker/haproxy/certs.d:/usr/local/etc/haproxy/certs.d \
  --net=internal --name haproxy_check haproxy:alpine -c -f /haproxy.cfg
```

### 3. Connect docker networks
```
docker network connect my_custom_network haproxy-certbot
```

### 4. Generate certificates
```
docker exec haproxy-certbot certbot-certonly \
  --domain example.com \
  --domain www.example.com \
  --email nmarus@gmail.com \
  --dry-run
```

### 5. Update certificates
```
docker exec haproxy-certbot haproxy-refresh
```

### 6. Check your browser

## Sample haproxy.cfg
```
global
  maxconn 1028

  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice

  ca-base /etc/ssl/certs
  crt-base /etc/ssl/private

  ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
  ssl-default-bind-options no-sslv3


defaults
  log global
  mode http

  option httplog
  option dontlognull

  timeout connect 5000
  timeout client 10000
  timeout server 10000

  default-server init-addr none


#enable resolving throught docker dns and avoid crashing if service is down while proxy is starting
resolvers docker_resolver
  nameserver dns 127.0.0.11:53


frontend http-in
  bind 0.0.0.0:80
  mode http

  reqadd X-Forwarded-Proto:\ http

  acl letsencrypt_http_acl path_beg /.well-known/acme-challenge/
  redirect scheme https if !letsencrypt_http_acl
  use_backend letsencrypt_http if letsencrypt_http_acl


frontend https-in
  bind *:443 ssl crt /usr/local/etc/haproxy/default.pem crt /usr/local/etc/haproxy/certs.d ciphers ECDHE-RSA-AES256-SHA:RC4-SHA:RC4:HIGH:!MD5:!aNULL:!EDH:!AESGCM
  mode http

  reqadd X-Forwarded-Proto:\ https

  use_backend my-default-backend


backend letsencrypt_http
  mode http
  server letsencrypt_http_srv 127.0.0.1:8080


backend my-default-backend
  mode http
  balance roundrobin
  server application_node_1 docker_container_name_1:80 check inter 5s resolvers docker_resolver resolve-prefer ipv4
  server application_node_2 docker_container_name_2:80 check inter 5s resolvers docker_resolver resolve-prefer ipv4
```
