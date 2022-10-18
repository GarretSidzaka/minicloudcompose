---
version: "3"

services:
  proxy:
    image: "nginxproxy/nginx-proxy:latest"
    container_name: nginx-proxy
    hostname: nginx-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true"
    volumes:
      - ${HOMEFOLDER}/certs:/etc/nginx/certs:ro
      - /etc/nginx/vhost.d
      - /usr/share/nginx/html
      - /var/run/docker.sock:/tmp/docker.sock:ro
    networks:
      - cloudnetwork
  letsencrypt:
    image: "jrcs/letsencrypt-nginx-proxy-companion:latest"
    container_name: cloud-letsencrypt
    hostname: letsencrypt
    restart: always
    volumes:
      - ${HOMEFOLDER}/certs:/etc/nginx/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
    volumes_from:
      - proxy:rw
    networks:
      - cloudnetwork
  mariadb:
    image: "mariadb:latest"
    container_name: mariadb
    hostname: mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${DBPASSWORD}
    volumes:
      - ${HOMEFOLDER}/mysql:/var/lib/mysql
    networks:
      - cloudnetwork
  minesworncom:
    image: "httpd"
    container_name: minesworncom
    hostname: minesworncom
    restart: always
    environment:
      - LETSENCRYPT_EMAIL=minesworncom@2enp.com
      - LETSENCRYPT_HOST=minesworn.com
      - VIRTUAL_HOST=minesworn.com
    volumes:
            - ${HOMEFOLDER}/minesworncom:/usr/local/apache2/htdocs:ro
    networks:
      - cloudnetwork
networks:
  cloudnetwork:
