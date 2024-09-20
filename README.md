# nextcloud-whiteboard-installation-guide
simple guide for the installation of the new nextcloud whiteboard app (docker + Apache Reverse proxy version)

based on: <https://github.com/nextcloud/whiteboard>

### get the image and run it in docker

- JWT_SECRET_KEY=some-random <- create a token and insert it
- NEXTCLOUD_URL=<https://nextcloud.local> <- put you nextcloud-domain here
- I don't know if this is needed, but I added "-p 127.0.0.1:3002:3002"

```
docker run -p 127.0.0.1:3002:3002 -e JWT_SECRET_KEY=some-random -e NEXTCLOUD_URL=https://nextcloud.local --restart unless-stopped -d ghcr.io/nextcloud-releases/whiteboard:v1.0.2
```

### Check Your firewall if needed

- Check Your firewall so it will allow Your Nextcloud to connect to the whiteboard-server

### Reverse proxy (Apache VirtualHost)

```
nano /etc/apache2/sites-available/yoursubdomain.conf
```

```
<VirtualHost *:80>
        ServerName whiteboard.domain.com
        ProxyPass /whiteboard http://localhost:3002/
        RewriteEngine on
        RewriteCond %{HTTP:Upgrade} websocket [NC]
        RewriteCond %{HTTP:Connection} upgrade [NC]
        RewriteRule ^/?whiteboard/(.*) "ws://localhost:3002/$1" [P,L]
</VirtualHost>
```

```
a2ensite yoursubdomain.conf
systemctl reload apache2
```

### Don't forget to create a subdomain

- whiteboard.domain.com with corresponding A- / AAAA-Record

### Certbot for SSL

```
certbot --apache
```

### Add the Whiteboard-App to Your Nextcloud

```
occ config:app:set whiteboard collabBackendUrl --value="http://nextcloud.local:3002"
occ config:app:set whiteboard jwt_secret_key --value="some-random"
```
