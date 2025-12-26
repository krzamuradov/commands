# Laravel settings API-only
```
// Создаём проект
composer create-project laravel/laravel ./ ^11
// Устанавливаем API
php artisan install:api
// Создаём middleware для JSON ответов.
php artisan make:middleware ForceJson
// Настройка CORS
php artisan config:publish cors
// bootstrap/app.php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\JsonResponse;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\HttpException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Illuminate\Auth\AuthenticationException;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        // Web-маршруты отключаем — никакого HTML
        web: null,

        // Только API-маршруты
        api: __DIR__ . '/../routes/api.php',

        // Консольные команды
        commands: __DIR__ . '/../routes/console.php',

        // Health-check (опционально)
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Подключаем API middleware
        $middleware->api(prepend: [
            \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        ]);

        // Убираем лишнее из web стека
        $middleware->web(remove: [
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        ]);
        $middleware->append(\Illuminate\Http\Middleware\HandleCors::class);

        $middleware->append(\App\Http\Middleware\ForceJson::class);

        // alias — если хотите использовать как именованный middleware
        $middleware->alias([
            'force.json' => \App\Http\Middleware\ForceJson::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Ошибки валидации
        $exceptions->render(function (ValidationException $e) {
            return new JsonResponse([
                'message' => 'Validation failed',
                'errors'  => $e->errors(),
            ], 422);
        });

        // Неавторизован
        $exceptions->render(function (AuthenticationException $e) {
            return new JsonResponse([
                'message' => 'Unauthenticated',
            ], 401);
        });

        // Доступ запрещён
        $exceptions->render(function (AccessDeniedHttpException $e) {
            return new JsonResponse([
                'message' => 'Forbidden',
            ], 403);
        });

        // Не найдено
        $exceptions->render(function (NotFoundHttpException $e) {
            return new JsonResponse([
                'message' => 'Api Not Found',
            ], 404);
        });

        // Любая другая HTTP ошибка
        $exceptions->render(function (HttpException $e) {
            return new JsonResponse([
                'message' => $e->getMessage() ?: 'HTTP Error',
            ], $e->getStatusCode());
        });

        // Все остальные ошибки (в т.ч. 500)
        $exceptions->render(function (Throwable $e) {
            return new JsonResponse([
                'message' => config('app.debug')
                    ? $e->getMessage()
                    : 'Server Error',
                'trace' => config('app.debug')
                    ? $e->getTrace()
                    : [],
            ], 500);
        });
    })
    ->create();
//Можно удалить
/routes/web.php
/resources/*
tailwindcss
vite.config.js
package.json
```
# LXC commands
```
// Создание контейнера UBUNTU
lxc launch ubuntu:22.04 ubuntu
```
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
sudo apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-tokenizer php-readline php-soap php-intl php-dom php-fileinfo php-opcache php-pgsql php-sqlite3 php-imap -y

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

# Полезные пакеты NPM
```
@vitejs/plugin-vue
vue
axios

bootstrap
import ""bootstrap/dist/css/bootstrap.min.css;
import ""bootstrap/dist/js/bootstrap.bundle.min.js;

vuex
vue-router
@fortawesome/fontawesome-free

bootstrap-icons
import "bootstrap-icons/font/bootstrap-icons.css";

flag-icons
import "flag-icons/css/flag-icons.min.css";

docx-preview
vue-i18n
pdf-vue3

dayjs
import dayjs from 'dayjs';
dayjs(meeting.meeting_at).format('DD.MM.YYYY')


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
//FOR LARAVEL
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
//FOR VUE ONLY
{
    "compilerOptions": {
      "baseUrl": "./",
      "paths": {
        "@/*": ["src/*"]
      },
      "module": "ESNext",
      "target": "ES6",
      "moduleResolution": "Node"
    },
    "exclude": ["node_modules", "public"],
    "include": ["src/**/*"]
  }
```
# VSCODE CONFIG
```
Ctrl + Shift + P → Preferences: Open Settings (JSON)

{
    // PHP — форматирует Intelephense
    "[php]": {
        "editor.defaultFormatter": "bmewburn.vscode-intelephense-client"
    },

    // Vue — форматирует Prettier
    "[vue]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },

    // JavaScript — тоже Prettier
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },

    // TypeScript (вдруг понадобится)
    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },

    // Общие настройки
    "editor.formatOnSave": true,

    "javascript.preferences.importModuleSpecifier": "non-relative",
    "typescript.preferences.importModuleSpecifier": "non-relative"
}



```
# PRETTIER CONFIG
```
// CREATE .prettierrc 

{
    "semi": true,
    "singleQuote": false,
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
## INERTIA SETTINGS
```
composer require inertiajs/inertia-laravel
php artisan inertia:middleware
npm install vue @vitejs/plugin-vue @inertiajs/vue3

import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap/dist/js/bootstrap.bundle.min.js"

createInertiaApp({
    resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
    setup({ el, App, props, plugin }) {
        createApp({ render: () => h(App, props) })
            .use(plugin)
            .mount(el);
    },
});

```

## RUN_DEV_SERVER
```
CREATE RUN_DEV_SERVER.bat

@echo off
cd /d "%~dp0"

start /b php artisan serve
start /b npm run dev
start /b code .
start "" "http://127.0.0.1:8000"

```

## Проверка доступа на JS
```
export const permissions = {
    1: ["list-meetings", "show-meeting", "create-meeting", "edit-meeting", "delete-meeting", "users-list", "users-create"],
    2: ["list-meetings", "show-meeting", "create-meeting", "edit-meeting", "delete-meeting", "users-list", "users-create"],
    3: ["list-meetings", "show-meeting", "list-documents", "show-document"],
};

export function can(ability) {
    try {
        const userRaw = localStorage.getItem("user");
        if (!userRaw) return false;

        const user = JSON.parse(userRaw);
        const userAbilities = permissions[user.role_id] || [];

        return userAbilities.includes(ability);
    } catch (e) {
        console.error("Ошибка в can():", e);
        return false;
    }
}
```

## Компонент DOCX-VIEWER
```
<script setup>
    import { ref, onMounted } from "vue";
    import { renderAsync } from "docx-preview";
    import { useRoute, useRouter } from "vue-router";

    const route = useRoute();
    const router = useRouter();
    console.log(route);

    const url = route.query.url;

    const container = ref(null);
    const styleContainer = ref(null);

    async function loadAndRenderDocx() {
        if (!container.value || !(container.value instanceof HTMLElement)) {
            console.error("container.value не HTMLElement");
            return;
        }
        if (!styleContainer.value || !(styleContainer.value instanceof HTMLElement)) {
            console.error("styleContainer.value не HTMLElement");
            return;
        }

        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error(`Ошибка загрузки: ${response.statusText}`);

            const arrayBuffer = await response.arrayBuffer();

            container.value.textContent = "";

            await renderAsync(arrayBuffer, container.value, styleContainer.value, {
                className: "docx-preview",
                inWrapper: false,
                experimental: true,
                ignoreWidth: true,
                ignoreHeight: true,
                renderChanges: true,
                renderAltChunks: true,
            });
        } catch (e) {
            console.error("Ошибка при рендеринге DOCX:", e);
        }
    }

    onMounted(() => {
        loadAndRenderDocx();
    });
</script>

<template>
    <button @click="router.back()" class="btn btn-outline-secondary mb-3"><i class="fa fa-reply"></i> Hujjatlar ro'yxatiga qaytish</button>
    <div class="d-flex justify-content-center align-items-center">
        <div ref="styleContainer" style="width: 0; height: 0; overflow: hidden; position: absolute; left: -9999px"></div>

        <div ref="container"></div>
    </div>
</template>

<style scoped>
    .docx-container .docx-preview {
        background-color: none !important;
        font-family: Arial, sans-serif;
        line-height: 1.5;
    }
    .docx-container [style*="background"] {
        background-color: white !important;
    }
</style>

```

## Компонент pdf-vue3
```
<script setup>
    import PDF from "pdf-vue3";
    import { useRoute, useRouter } from "vue-router";

    const route = useRoute();
    const router = useRouter();
    const url = route.query.url;
    console.log(url);

    const props = defineProps({
        url: { type: String },
    });
</script>

<template>
    <div class="container mt-4">
        <button @click="router.back()" class="btn btn-outline-secondary mb-3"><i class="fa fa-reply"></i> Hujjatlar ro'yxatiga qaytish</button>
        <PDF :src="url" pdfWidth="70%" scrollThreshold="10" />
    </div>
</template>

```

## Компонент Input
```
<script setup>
    const props = defineProps({
        type: { type: String, default: "text" },
        name: { type: String, required: true },
        label: { type: String, required: true },
        is_invalid: { type: String, default: "" },
    });

    const model = defineModel();
</script>

<template>
    <div class="position-relative">
        <input :type="props.type" :name="props.name" :id="props.name" class="form-control" v-model="model" :class="{ 'is-invalid': props.is_invalid }" v-bind="$attrs" />
        <label :for="props.name" class="position-absolute top-0 start-0 translate-middle-y bg-white px-1 ms-3 small text-secondary rounded-0">{{ props.label }}</label>
        <div class="invalid-feedback" v-if="props.is_invalid">{{ props.is_invalid }}</div>
    </div>
</template>

<style scoped>
    input {
        height: 3rem;
    }
    input:focus {
        border-color: #0d6efd !important; /* или любой другой */
        box-shadow: 0 0 3px rgb(50, 83, 231); /* стандарт bootstrap */
    }

    input:-webkit-autofill {
        box-shadow: 0 0 0px 1000px white inset !important;
        -webkit-box-shadow: 0 0 0px 1000px white inset !important;
        -webkit-text-fill-color: #000 !important;
    }

    .is-invalid {
        box-shadow: none !important;
    }
</style>

```

## Компонент Select
```
<script setup>
    const props = defineProps({
        type: { type: String, default: "text" },
        name: { type: String, required: true },
        label: { type: String, required: true },
        is_invalid: { type: String, default: "" },
        is_required: { type: Boolean, default: true },
    });

    const model = defineModel();
</script>

<template>
    <div class="position-relative">
        <select :name="props.name" :id="props.name" class="form-select" v-model="model" :class="{ 'is-invalid': props.is_invalid }" v-bind="$attrs">
            <slot></slot>
        </select>
        <label :for="props.name" class="position-absolute top-0 start-0 translate-middle-y bg-white px-1 ms-3 small text-secondary rounded-0">
            {{ props.label }}
            <span v-if="props.is_required" class="text-danger">*</span>
        </label>
        <div class="invalid-feedback" v-if="props.is_invalid">{{ props.is_invalid }}</div>
    </div>
</template>

<style scoped>
    select {
        height: 3rem;
    }
    select:focus {
        border-color: #417cd4 !important; /* или любой другой */
        box-shadow: 0 0 3px rgb(50, 83, 231); /* стандарт bootstrap */
    }

    select:-webkit-autofill {
        box-shadow: 0 0 0px 1000px white inset !important;
        -webkit-box-shadow: 0 0 0px 1000px white inset !important;
        -webkit-text-fill-color: #000 !important;
    }

    .is-invalid {
        box-shadow: none !important;
    }
</style>


```
## Компонент Navbar
```
<script setup></script>

<template>
    <nav class="navbar navbar-expand-lg bg-white shadow-sm px-4 mb-3">
        <a class="navbar-brand fw-bold" href="/">
            <img src="@/assets/logo.png" alt="Logo" width="100px" />
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarContent">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item">
                    <a class="nav-link active" href="#">Главная</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#">О нас</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#">Услуги</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#">Контакты</a>
                </li>
            </ul>
        </div>
    </nav>
</template>

<style scoped>
    nav {
        min-height: 100px;
    }
    .navbar-nav .nav-link {
        position: relative;
        padding: 0.5rem 1rem;
        color: #333;
        font-weight: 500;
        transition: all 0.3s ease;
    }

    .navbar-nav .nav-link::after {
        content: "";
        position: absolute;
        left: 1rem;
        bottom: 0;
        width: 0;
        height: 2px;
        background-color: #0d6efd; /* основной цвет Bootstrap */
        transition: width 0.3s ease;
    }

    .navbar-nav .nav-link:hover::after {
        width: calc(100% - 2rem); /* анимация underline */
    }

    .navbar-nav .nav-link:hover {
        color: #0d6efd;
    }

    .navbar-nav .nav-link.active {
        color: #0d6efd;
    }

    .navbar-nav .nav-link.active::after {
        width: calc(100% - 2rem);
    }
</style>

```
## Компонент loader
```
<script setup></script>

<template>
    <div class="d-flex justify-content-center align-items-center position-absolute top-0 start-0 vh-100 w-100 bg-white z-3">
        <img src="@/assets/logo.png" alt="Logo" width="150" class="animated-logo" />
    </div>
</template>

<style scoped>
    .animated-logo {
        animation: pulse 0.9s infinite ease-in-out;
    }

    @keyframes pulse {
        0%,
        100% {
            transform: scale(0.8);
        }
        50% {
            transform: scale(1.1);
        }
    }
</style>


```

## Настройка SPA приложения в подкаталоге.
```
//vite.config.js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import laravel from "laravel-vite-plugin";

export default defineConfig({
    base: "/appname/build/",
    plugins: [
        laravel({
            input: ["resources/css/app.css", "resources/js/app.js"],
            refresh: true,
            buildDirectory: "build",
            manifest: true,
        }),
        vue(),
    ],
});

//nginx in container
server {
    listen 80;
    server_name localhost;

    root /srv/appname/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot|otf)$ {
        expires 7d;
        access_log off;
        add_header Cache-Control "public";
    }
}

nginx in host
    location /sms/ {
        proxy_pass http://container-ip/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
//router.js
const router = createRouter({
    history: createWebHistory("/appname/"),
    routes,
});
.env
APP_URL=https://domain.com/appname
ASSET_URL=https://domain.com/appname
VITE_API_URL=https://domain.com/appname/api
```

## LARAVEL WORKER SETTINGS
```
//-----------INSTALL SUPERVISOR
sudo apt update && sudo apt install supervisor -y
systemctl status supervisor
sudo nano /etc/supervisor/conf.d/laravel-worker.conf

//------------CONFIG
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/your-app/artisan queue:work --sleep=3 --tries=3 --timeout=90
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/your-app/storage/logs/worker.log
stopwaitsecs=3600
//----------PARAMS
--sleep=3 → если нет задач, ждёт 3 секунды.

--tries=3 → если задача упала, Laravel попробует 3 раза.

--timeout=90 → если job выполняется дольше 90 сек → прервётся.

numprocs=1 → один процесс (можно увеличить для высокой нагрузки).

//----------START SERVICE
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
sudo supervisorctl status

//---------- STOP AND DELETE SERVICE

sudo supervisorctl stop laravel-worker:*
sudo rm /etc/supervisor/conf.d/laravel-worker.conf
sudo supervisorctl reread
sudo supervisorctl update

sudo systemctl stop supervisor 
```

## REDIS SETTINGS

```
sudo apt update
sudo apt install redis-server -y
sudo systemctl status redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server

sudo mcedit /etc/redis/redis.conf
bind 0.0.0.0
requirepass q1w2e3
supervised systemd
sudo systemctl restart redis-server
//----------- Проверка
redis-cli
127.0.0.1:6379> ping
127.0.0.1:6379> auth q1w2e3
OK
127.0.0.1:6379> ping
PONG

composer require predis/predis

//---------- .env
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

php artisan cache:clear
php artisan config:clear
php artisan queue:failed

```
## PORT FORWARD

```
POWERSHELL
-------ADD FORWARD
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport={PORT} connectaddress={ADDRESS_IN_WSL} connectport={PORT_IN_WSL}
-------SHOW FORWARDS
netsh interface portproxy show all
-------DELETE FORWARDS
netsh interface portproxy delete v4tov4 listenport=8000 listenaddress=0.0.0.0

```

