## HyreTalenta

HyreTalenta is a apprentice search platform

Link App:
[DevHyre](http://dev.hyre.co.id)
[API-DevHyre](http://api-dev.hyre.co.id)

### Documentation

---

Server use operating system **Ubuntu 22.04**

**IP Local** : 172.31.38.92

**IP Public** : 43.218.129.226

**Server Application and Version**

```bash
# ufw
0.36.1

# nginx
nginx/1.18.0

# git
2.34.1

# php
8.1.2-1ubuntu2.11

# nodejs
v18.16.0

# npm
9.6.6

# composer
2.2.6

# pm2
5.3.0

# postgre
14.7
```

**UFW Status**

```bash
ubuntu@ip-172-31-38-92:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)

# in this case i only allowed OpenSSH and Nginx Full app
```

**Cloning repo from github**

```bash
# change directory
ubuntu@ip-172-31-38-92:~$ cd /var/www/
# cloning repository
ubuntu@ip-172-31-38-92:/var/www/$ sudo git clone https://github.com/devopsvapecorp/hyre-web.git
# change directory
ubuntu@ip-172-31-38-92:/var/www/$ cd hyre-web/
# basic git config
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ sudo git config --global user.name arifsptrra
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ sudo git config --global user.email arif.saputra@vapecorp.id
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ sudo git config --global safe.directory /var/www/hyre-web
```

**Cloning laravel project**

```bash
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ composer install
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ sudo cp .emv.example .env
ubuntu@ip-172-31-38-92:/var/www/hyre-web/$ sudo php artisan key:generate
```

**NPM Install to frontend app**

```bash
# change directory
ubuntu@ip-172-31-38-92:~$ cd /var/www/hyre-web/frontend/
# install dependencies and build
ubuntu@ip-172-31-38-92:/var/www/hyre-web/frontend$ npm install
ubuntu@ip-172-31-38-92:/var/www/hyre-web/frontend$ npm run build
```

**PM2 Configuration**

```bash
# change directory
ubuntu@ip-172-31-38-92:~$ cd /var/www/hyre-web/frontend/
# make pm2 to run project in frontend directory
ubuntu@ip-172-31-38-92:/var/www/hyre-web/frontend$ sudo pm2 start npm --name dev-hyre -- run dev
# check pm2
ubuntu@ip-172-31-38-92:/var/www/hyre-web/frontend$ sudo pm2 list
┌────┬─────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name        │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼─────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 3  │ dev-hyre    │ default     │ N/A     │ fork    │ 60005    │ 12h    │ 0    │ online    │ 0%       │ 66.5mb   │ root     │ disabled │
└────┴─────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

**NGINX Configuration**

```bash
# configuration for domain dev.hyre.co.id
# change directory
ubuntu@ip-172-31-38-92:~$ cd /etc/nginx/sites-available/
# copy file
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo cp default dev.hyre.co.id
# edit file dev.hyre.co.id
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo nano dev.hyre.co.id

# content of the dev.hyre.co.id
server {
        listen 80;
        listen [::]:80;

				# root to
        root /var/www/hyre-web/frontend;

				# this is file name to execute
        index index.html index.htm index.nginx-debian.html;

				# for server name
        server_name dev.hyre.co.id www.dev.hyre.co.id;

        location / {
								# port must match if npm run dev
                proxy_pass http://localhost:5173;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}

# make soft link for enable sites
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/dev.hyre.co.id /etc/nginx/sites-enabled/
```

```bash
# configuration for domain api-dev.hyre.co.id
# change directory
ubuntu@ip-172-31-38-92:~$ cd /etc/nginx/sites-available/
# copy file
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo cp default api-dev.hyre.co.id
# edit file api-dev.hyre.co.id
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo nano api-dev.hyre.co.id

# content of the api-dev.hyre.co.id
server {
        listen 80;

				# for server name
        server_name api-dev.hyre.co.id;

				# root to
        root /var/www/hyre-web/public;

				# this is file name to execute
        index index.php index.html index.htm;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-Content-Type-Options "nosniff";

        index index.html;

        charset utf-8;

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }

				location = /favicon.ico { access_log off; log_not_found off; }
				location = /robots.txt { access_log off; log_not_found off; }

				error_page 404 /index.php;

				location ~ \.php$ {
					      try_files $uri =404;
					      fastcgi_split_path_info ^(.+\.php)(/.+)$;
	              fastcgi_pass unix:/run/php/php8.1-fpm.sock;
				        fastcgi_index index.php;
				        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
				        include fastcgi_params;
				}

        location ~ /\.ht {
                deny all;
        }

        access_log /var/log/nginx/laravel-access.log;
        error_log /var/log/nginx/laravel-error.log;
}

# make soft link for enable sites
ubuntu@ip-172-31-38-92:/etc/nginx/sites-available$ sudo ln -s /etc/nginx/sites-available/api-dev.hyre.co.id /etc/nginx/sites-enabled/
```
