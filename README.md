# HAProxy with Certbot
Based on [nmarus/docker-haproxy-certbot](https://github.com/nmarus/docker-haproxy-certbot)
For detailed info see origin repo.

## Changes
* Moved to Alpine Linux
* Supervisord logging
* Updated CertBot to >=0.22
* Updated HAProxy to 1.8.8
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
  -v /docker/haproxy/logs:/var/log/supervisord \
  dock1100/haproxy-certbot:latest
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

## Sample configs
For sample configs check `haproxy.sample.cfg` and `docker-compose.sample.cfg`