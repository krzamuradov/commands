
# BASE DEPENDENCIES
```
apt install mc curl git zip unzip openssh-server -y
```

# NGINX
```
apt install nginx -y
```
### NGINX CONF FOR BACKEND AND FRONTEND
```
server {
    listen 80;
    server_name //IP OR DOMAIN NAME;

    root /srv/PROJECT_NAME/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }


    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|ttf|woff|woff2|eot)$ {
        root /srv/PROJECT_NAME/frontend/dist;
        expires 30d;
        add_header Cache-Control "public";
    }

    location /api/ {
        root /srv/PROJECT_NAME/backend/public;
        try_files $uri $uri/ /index.php?$query_string;

        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ \.php$ {
        root /srv/PROJECT_NAME/backend/public;
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
### NGINX FOR DEFAULT SERVER
```
server {
    listen 80 default_server;
    server_name _;
    index index.html;
    root /var/www/html;
    #return 403;
}
```
### NGINX PROXY CONF
```
server {
    listen 80;
    server_name //IP OR DOMAIN;

    location / {
        proxy_pass http://0.0.0.0;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
### NGINX CONF FOR ONLY PHP PROJECT
```
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
```

# PHP
```
apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath -y
```
# COMPOSER
```
curl -s https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

# MARIADB
```
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
```

# PHPMYADMIN
```
apt install phpmyadmin
ln -s /usr/share/phpmyadmin /var/www/
```    

# NPM
```
apt install npm -y
npm install
npm install vue @vitejs/plugin-vue bootstrap vuex vue-router @fortawesome/fontawesome-free
```
# NPM publish package
```
npm init -y
npm login
npm publish
npm verison patch
npm publish --access=public
```

# LARAVEL INSTALL
```
composer create-project --prefer-dist laravel/laravel ./ ^10
```


# Bash                       
```
mcedit ~/.bashrc
source ~/.bashrc -                                
cat ~/.bash_history -----                             
which package -----                
df -h               
du -sm /var/lib/lxd/containers/name ---                  
iptables -nvL -t nat -                  

sudo sshfs -o allow_other root@10.96.222.13:/var/www/html /mnt/projects/crm/
```
# SQLITE3

> [!NOTE]
> Нужно раскомментировать в php.ini
> ;extension=pdo_sqlite
> ;extension=sqlite3
```
sudo apt-get update
sudo apt-get install php-sqlite3
php -m | grep sqlite  проверка sqlite
```


# GIT commands
```
git clone
git pull
git init
git add .
git commit -m "message"
git remote add origin https://github.com/
git push
git cherry-pick
git branch
git checkout

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)" ssh-add ~/.ssh/id_rsa

cat ~/.ssh/id_rsa.pub
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
git config --global user.name "Ваше Имя"
git config --global user.email "your_email@example.com"

```



# Vuetify install example
```
npm install vuetify@next sass sass-loader @mdi/font
```
```
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
```

# PHP PROD SERVICE PROVIDER
```
class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        if($this->app->environment('production')){
            URL::forceScheme('https');
        }
    }
```

# LARAVEL VALIDATION CUSTOM RESPONSE
```

use Illuminate\Contracts\Validation\Validator;
use Illuminate\Http\JsonResponse;
use Illuminate\Validation\ValidationException;

 protected function failedValidation(Validator $validator)
    {
        $response = new JsonResponse([
            'errors' => $validator->errors(),
        ], 422);

        throw new ValidationException($validator, $response);
    }
```

# VSCODE SETTINGS JS IN LARAVEL PROJECT 
```
// CREATE jsconfig.json

{
    "compilerOptions": {
      "baseUrl": "./",
      "paths": {
        "@/*": ["resources/js/*"]
      },
      "module": "ESNext",
      "target": "ES6",
      "moduleResolution": "Node"
    },
    "exclude": ["node_modules", "public"],
    "include": ["resources/js/**/*"]
  }

CTRL+, 
"javascript.preferences.importModuleSpecifier": "non-relative",
"typescript.preferences.importModuleSpecifier": "non-relative"


```

# PRETTIER CONFIG
```
// CREATE .prettierrc 

{
    "semi": true,
    "singleQuote": true,
    "tabWidth": 4,
    "useTabs": false,
    "trailingComma": "es5",
    "bracketSpacing": true,
    "arrowParens": "always",
    "printWidth": 170,
    "vueIndentScriptAndStyle": true
}

CREATE .prettierignore
node_modules/
dist/
public/


```

# COMPOSER PLUGIN FOR INTERFACE SERVICE 
```
composer require theanik/laravel-more-command --dev
```

# DATE FORMAT JAVASCRIPT 
```
const isoDate = "2024-10-30T12:30:44.000000Z";
const date = new Date(isoDate);

const options = {
  year: 'numeric',
  month: '2-digit',
  day: '2-digit',
  hour: '2-digit',
  minute: '2-digit',
  second: '2-digit',
  hour12: false, // 24-часовой формат
  timeZone: 'UTC'
};

const formattedDate = new Intl.DateTimeFormat('ru-RU', options).format(date);
console.log(formattedDate); 
// Результат: 30.10.2024, 12:30:44

```

# LARAVEL CORS CONFIG
```
php artisan config:publish cors

return [
       'paths' => ['*'],
       'allowed_methods' => ['GET', 'POST', 'PUT', 'OPTIONS'],
       'allowed_origins' => ['https://localhost82'],
       'allowed_origins_patterns' => [],
       'allowed_headers' => ['Origin', 'Content-Type', 'X-Auth-Token', 'Cookie'],
       'exposed_headers' => [],
       'max_age' => 0,
       'supports_credentials' => true,
];

php artisan config:clear
php artisan route:clear
php artisan cache:clear

```

# FILE UPLOAD SIZE
## NGINX
```
client_max_body_size 20M;
```
## PHP
```
///etc/php/8.2/fpm/php.ini
post_max_size = 20M
upload_max_filesize = 20M
```
```
#bash
sudo systemctl restart php8.2-fpm
sudo systemctl reload nginx
```

