```
apt install mc curl git zip unzip openssh-server -y
```
```
server {
    listen 80;
    server_name 10.96.222.173;

    root /srv/normativ/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }


    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|ttf|woff|woff2|eot)$ {
        root /srv/normativ/frontend/dist;
        expires 30d;
        add_header Cache-Control "public";
    }

    location /api/ {
        root /srv/normativ/backend/public;
        try_files $uri $uri/ /index.php?$query_string;

        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ \.php$ {
        root /srv/normativ/backend/public;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # Логи
    access_log /var/log/nginx/vue_access.log;
    error_log /var/log/nginx/vue_error.log;
}
```


#NGINX
apt install nginx -y

server {
    listen 80 default_server;
    server_name _;
    index index.html;
    root /var/www/html;
    #return 403;
}

server {
    listen 80;
    server_name normativ.post.uz;

    location / {
        proxy_pass http://10.158.127.132;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    index index.php
    server_name localhost;
    index index.php;
    root /var/www/html/public;

    location ~ \.php$ {
	try_files $uri =404;
        include fastcgi_params;
	fastcgi_index index.php;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

	location / {
        	try_files $uri $uri/ /index.php$is_args$args;
    	}
}

#PHP
apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath -y
#COMPOSER
curl -s https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

#MARIADB
apt install mariadb-server -y

systemctl stop mariadb
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables --skip-networking"
systemctl start mariadb
systemctl status mariadb
mysql -u root
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
systemctl unset-environment MYSQLD_OPTS
systemctl restart mariadb

#PHPMYADMIN

apt install phpmyadmin
ln -s /usr/share/phpmyadmin /var/www/
                    nginx                 

#NPM
apt install npm -y
npm install
npm install vue @vitejs/plugin-vue bootstrap vuex vue-router @fortawesome/fontawesome-free

#LARAVEL INSTALL

composer create-project --prefer-dist laravel/laravel ./ ^10



#                             

mcedit ~/.bashrc
source ~/.bashrc -                                
cat ~/.bash_history -----                             
which package -----                
df -h               
du -sm /var/lib/lxd/containers/name ---                  
iptables -nvL -t nat -                  


sudo sshfs -o allow_other root@10.96.222.13:/var/www/html /mnt/projects/crm/

#SQLITE3
sudo apt-get update
sudo apt-get install php-sqlite3

php -m | grep sqlite  проверка sqlite

extension=pdo_sqlite
extension=sqlite3

#GIT
git clone
git pull
git init
git add .
git commit -m "message"
git remote add origin https://github.com/
git push


#vuetify
npm install vuetify@next sass sass-loader @mdi/font

// Импорт Vuetify
import { createVuetify } from 'vuetify';
import 'vuetify/styles'; // Стили Vuetify
import '@mdi/font/css/materialdesignicons.css'; // Иконки

// Импорт компонентов и директив
import * as components from 'vuetify/components';
import * as directives from 'vuetify/directives';

// Инициализация Vuetify с темой по умолчанию
const vuetify = createVuetify({
  components,
  directives,
  theme: {
    defaultTheme: 'light',
  },
});

createApp(App).use(vuetify).mount('#app');
