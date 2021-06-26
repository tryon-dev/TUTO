## Comment installer Pufferpanel

**PufferPanel est prêt à l'emploi avec la prise en charge des distributions suivantes :**

- **Ubuntu 20.04**

- **Ubuntu 18.04**

- **Ubuntu 16.04**

- **Debian 10**

- **Debian 9**

- **Debian 8**

- **CentOS 7**

- **Raspbian 10**



Pour une installation plus simple, si vous possédez l'une des distributions prises en charge, il vous suffit d'installer notre package et de commencer !



**Ubuntu/Debian :**

```
curl -s https://packagecloud.io/install/repositories/pufferpanel/pufferpanel/script.deb.sh | sudo bash
```

```
sudo apt-get install pufferpanel
```

```
systemctl enable pufferpanel
```



**CentOS :**

```
curl -s https://packagecloud.io/install/repositories/pufferpanel/pufferpanel/script.rpm.sh | sudo bash
```

```
yum install pufferpanel
```

```
systemctl enable pufferpanel
```





**Les ports suivants sont utilisés par PufferPanel :**

8080 : Accès Internet

5657 : SFTP

Veuillez autoriser le trafic vers/depuis ces ports pour utiliser pleinement votre installation.







**Ajout d'un administrateur**
Pour créer votre premier utilisateur, exécutez la commande suivante. Assurez-vous d'entrer « O » lorsqu'il vous demande s'il s'agit d'un administrateur afin que vous puissiez utiliser pleinement votre panneau.

```
pufferpanel user add
```



Démarrage du panneau :

```
systemctl start pufferpanel
```



**Nginx**

Configurer Nginx pour servir votre panel est assez simple. Tout ce que vous avez à faire est de configurer un bloc proxy_pass pour lui transmettre tout le trafic destiné au panneau.

```
server {
  listen 80;
  server_name panel.examplehost.com;

  location ~ ^/\.well-known {
    root /var/www/html;
    allow all;
  }

  location / {
      return 301 https://$host$request_uri;
  }
}

server {
  listen 443 ssl;
  root /var/www/pufferpanel;

  ssl_certificate     /etc/nginx/ssl/<server>.crt;
  ssl_certificate_key /etc/nginx/ssl/<server>.key;

  server_name panel.examplehost.com;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Nginx-Proxy true;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://localhost:8080;
  }
}
```

Et sans le SSL

```
server {
    listen 80;
    root /var/www/pufferpanel;

    server_name panel.examplehost.com;

    location ~ ^/\.well-known {
      root /var/www/html;
      allow all;
    }

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }
}
```



**Apache**

Les exemples suivants montrent comment configurer un vhost pour rendre une instance PufferPanel sur le même serveur accessible sous panel.example.com

Pour utiliser Apache 2 comme proxy inverse, vous devrez activer les modules suivants :

mod_proxy

mod_proxy_http

mod_proxy_wstunnel

mod_rewrite

```
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerName panel.example.com

        ProxyPreserveHost On
        SSLProxyEngine On
        ProxyPass / http://localhost:8080/
        ProxyPassReverse / http://localhost:8080/

        RewriteEngine on
        RewriteCond %{HTTP:Upgrade} websocket [NC]
        RewriteCond %{HTTP:Connection} upgrade [NC]
        RewriteRule .* ws://localhost:8080%{REQUEST_URI} [P]

        SSLEngine on
        SSLCertificateFile /etc/letsencrypt/live/panel.example.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/panel.example.com/privkey.pem
    </VirtualHost>
</IfModule>
```

Et sans le SSL

```
<VirtualHost _default_:80>
    ServerName panel.example.com

    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/

    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule .* ws://localhost:8080%{REQUEST_URI} [P]
</VirtualHost>
```



**FINI**

Et c'est tout! Votre panneau est maintenant disponible sur le port 8080 de votre serveur.

