# НАСТРОЙКИ Laravel ПОД API-only
#### Создаём проект
```
composer create-project laravel/laravel ./ ^11
```
#### В LARAVEL 11+ НЕ ПРЕДУСТАНОВЛЕНА РАБОТА С API ДЛЯ УСТАНОВКИ
```
php artisan install:api
```
#### СОЗДАЁМ MIDDLEWARE ДЛЯ ТОГО ЧТОБЫ ОТВЕТЫ ВСЕГДА БЫЛИ В ВИДЕ JSON
```
php artisan make:middleware ForceJson
```
##### КОД MIDDLEWARE FORCEJSON
```
class ForceJson
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next)
    {
        $request->headers->set('Accept', 'application/json');
        $response = $next($request);
        $response->headers->set('Content-Type', 'application/json');
        return $response;
    }
}
```
#### УСТАНОВКА КОНФИГА CORS
```
php artisan config:publish cors
```
```
return [
       'paths' => ['*'],
       'allowed_methods' => ['GET', 'POST', 'PUT', 'OPTIONS'],
       'allowed_origins' => ['https://localhost82'], // РАЗРЕШЁННЫЕ АДРЕСА
       'allowed_origins_patterns' => [],
       'allowed_headers' => ['Origin', 'Content-Type', 'X-Auth-Token', 'Cookie'],
       'exposed_headers' => [],
       'max_age' => 0,
       'supports_credentials' => true,
];
```
```
php artisan config:clear
```
```
php artisan route:clear
```
```
php artisan cache:clear
```
##### КОД /bootstrap/app.php
```
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
```
> [!NOTE]
> Можно удалить /routes/web.php /resources/* tailwindcss vite.config.js package.json
# LXC commands
```
// Создание контейнера UBUNTU
lxc launch ubuntu:22.04 ubuntu
```
# НАСТРОЙКА ЧИСТОГО СЕРВЕРА
### БАЗОВЫЕ УСТАНОВКИ
```
apt install mc curl git zip unzip openssh-server -y
```
### УСТАНОВКА PHP
```
sudo apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-tokenizer php-readline php-soap php-intl php-dom php-fileinfo php-opcache php-pgsql php-sqlite3 php-imap -y
```
### УСТАНОВКА COMPOSER
##### СКАЧИВАНИЕ COMPOSER
```
curl -s https://getcomposer.org/installer | php
```
##### ПЕРЕМЕЩЕНИЕ COMPOSER
```
mv composer.phar /usr/local/bin/composer
```
### УСТАНОВКА NODE.JS
##### Добавляем репозиторий Node.js (например, LTS 20)
```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```
##### УСТАНОВКА NODE.JS (вместе с NPM)
```
apt install -y nodejs
```

##### ПРОВЕРКА
```
node -v
npm -v
```

# УСТАНОВКА И НАСТРОЙКА MARIADB
##### УСТАНОВКА
```
apt install mariadb-server -y
```
##### НАСТРОЙКА
```
systemctl stop mariadb
```
```
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables --skip-networking"
```
```
systemctl start mariadb
```
```
systemctl status mariadb
```
```
mysql -u root
```
```
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```
```
systemctl unset-environment MYSQLD_OPTS
```
```
systemctl restart mariadb
```
# УСТАНОВКА PHPMYADMIN
```
apt install phpmyadmin -y
```
```
ln -s /usr/share/phpmyadmin /var/www/
```
# УСТАНОВКА И НАСТРОЙКА SQLITE3
#### УСТАНОВКА
```
apt-get install php-sqlite3
```
> [!NOTE]
> Нужно раскомментировать в php.ini
> ;extension=pdo_sqlite
> ;extension=sqlite3

#### ПРОВЕРКА
```
php -m | grep sqlite
```

# УСТАНОВКА И НАСТРОЙКА NGINX
#### УСТАНОВКА
```
apt install nginx -y
```
#### КОНФИГ NGINX ЕСЛИ ПРОЕКТ НЕ МОНОЛИТ BACKEND И FRONTEND 80 ПОРТ
```
server {
    listen 80;
    server_name <IP_OR_DOMAIN_NAME>;
	#FRONTEND
    root /<FRONTEND_FOLDER>/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|ttf|woff|woff2|eot)$ {
        root /<FRONTEND_FOLDER>/dist;
        expires 30d;
        add_header Cache-Control "public";
    }

	# BACKEND
    location /api/ {
        root /<BACKEND_FOLDER>/public;
        try_files $uri $uri/ /index.php?$query_string;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ \.php$ {
        root /<BACKEND_FOLDER>/public;
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
#### НАСТРОЙКА NGINX ЕСЛИ НЕ СОВПАЛ ДОМЕН ИЛИ ЗАХОДЯТ ПО IP
```
server {
    listen 80 default_server;
    server_name _;
    index index.html;
    root /var/www/html;
    #return 403;
}
```
#### НАСТРОЙКА NGINX PROXY
```
server {
    listen 80;
    server_name <IP_OR_DOMAIN_NAME>;

    location / {
        proxy_pass http://<IP_ADRESS>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
#### НАСТРОЙКА NGINX ДЛЯ ПРОЕКТА НА LARAVEL БЕЗ VUE
```
server {
    listen 80;
    server_name <IP_OR_DOMAIN_NAME>;

	root /<BACKEND_FOLDER>/public;
    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri =404;

        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;

        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;

        fastcgi_index index.php;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```
# НАСТРОКА РАЗМЕРА ЗАГРУЖАЕМОГО НА СЕРВЕР ФАЙЛА
#### NGINX
```
client_max_body_size 20M;
```
#### PHP
>[!NOTE]
> РЕДАКТИРУЕМ ФАЙЛ /etc/php/8.2/fpm/php.ini
```
post_max_size = 20M
upload_max_filesize = 20M
```
###### ПЕРЕЗАПУСК
```
sudo systemctl restart php8.2-fpm
```
```
sudo systemctl reload nginx
```

# ПОЛЕЗНЫЕ ПАКЕТЫ NPM
#### УСТАНОВКА БАЗОВЫХ ПАКЕТОВ ОДНОЙ КОМАНДОЙ В МОНОЛИТНОМ ПРОЕКТЕ
```
npm install vue @vitejs/plugin-vue vue-router pinia bootstrap bootstrap-icons vue-i18n dayjs
```
> [!NOTE]
> ТАКЖЕ ЕСТЬ
> @fortawesome/fontawesome-free
> flag-icons
> docx-preview
> pdf-vue3

###### ПРИМЕРЫ ПОДКЛЮЧЕНИЯ

```
#-----BOOTSTRAP
import ""bootstrap/dist/css/bootstrap.min.css;
import ""bootstrap/dist/js/bootstrap.bundle.min.js;

#-----BOOTSTRAP-ICONS
import "bootstrap-icons/font/bootstrap-icons.css";

#-----FLAG-ICONS
import "flag-icons/css/flag-icons.min.css";

#-----DAYJS
import dayjs from 'dayjs';
dayjs(<ISO_DATE>).format('DD.MM.YYYY')

```
# NPM publish package
```
npm init -y
npm login
npm publish
npm verison patch
npm publish --access=public
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

# УСТАНОВКА И НАСТРОЙКА Vuetify
##### УСТАНОВКА
```
npm install vuetify@next sass sass-loader @mdi/font
```
#### ПОДКЛЮЧЕНИЕ В app.js
```
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

# SERVICE PROVIDER ДЛЯ PRODUCTION ЧТОБЫ ВСЕГДА ИСПОЛЬЗОВАЛСЯ HTTPS
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

# ОТПРАВКА СВОЕГО СООБЩЕНИЯ ПРИ ВАЛИДАЦИИ
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

# НАСТРОЙКИ VSCODE
#### АВТОЗАГРУЗКА ФАЙЛОВ И КОМПОНЕНТОВ
> [!NOTE]
> СОЗДАЁМ В КОРНЕ ФАЙЛ jsconfig.json

###### ДЛЯ МОНОЛИТНОГО ПРОЕКТА
```

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
```
###### ДЛЯ ПРОЕКТА НА VUE.JS
```
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
# НАСТРОЙКА PRETTIER
> [!NOTE]
> СОЗДАЁМ В КОРНЕ ФАЙЛ .prettierrc
```
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
```
> [!NOTE]
> СОЗДАЁМ В КОРНЕ ФАЙЛ .prettierignore
```
node_modules/
dist/
public/

```

# COMPOSER PLUGIN FOR INTERFACE SERVICE 
> [!NOTE]
> ПАКЕТ ДЛЯ ТОГО ЧТОБЫ МОЖНО БЫЛО ПИСАТЬ СРАЗУ
> php artisan make:interface,php artisan make:service
```
composer require theanik/laravel-more-command --dev
```

# ФОРМАТИРОВАНИЕ ДАТЫ НА ЧИСТОМ JAVASCRIPT 
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


# НАСТРОЙКИ INERTIA.JS
#### УСТАНОВКА
```
composer require inertiajs/inertia-laravel
```
```
php artisan inertia:middleware
```
```
npm install vue @vitejs/plugin-vue @inertiajs/vue3
```
#### ПОДКЛЮЧЕНИЕ
```
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
    resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
    setup({ el, App, props, plugin }) {
        createApp({ render: () => h(App, props) })
            .use(plugin)
            .mount(el);
    },
});
```

## ПРИМЕР ПРОВЕРКИ ДОСТУПА НА JAVASCRIPT
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

# НАСТРОЙКА SPA ПРИЛОЖЕНИЯ ЕСЛИ ДОМЕН ОДИН А ПРИЛОЖЕНИИ НЕСКОЛЬКО
> [!NOTE]
> ФАЙЛ vite.config.js
```
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import laravel from "laravel-vite-plugin";

export default defineConfig({
    base: "/<APP_NAME>/build/",
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
```
> [!NOTE]
> ФАЙЛ router.js
```
const router = createRouter({
    history: createWebHistory("/<APP_NAME>/"),
    routes,
});
```
> [!NOTE]
> ФАЙЛ .env
```
APP_URL=https://<DOMAIN>/<APP_NAME>
ASSET_URL=https://<DOMAIN>/<APP_NAME>
VITE_API_URL=https://<DOMAIN>/<APP_NAME>/api
```
#### НАСТРОЙКА NGINX
```
location /sms/ {
	proxy_pass http://<CONTAINER_IP>/; // ЕСЛИ LXC
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
}
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

## УСТАНОВКА И НАСТРОЙКА REDIS

```
sudo apt update
```
```
sudo apt install redis-server -y
```
```
sudo systemctl status redis-server
```
```
sudo systemctl start redis-server
```
```
sudo systemctl enable redis-server
```
#### НАСТРОЙКА НА СЕРВЕРЕ
> [!NOTE]
> ФАЙЛ /etc/redis/redis.conf
```
bind 0.0.0.0
requirepass <PASSWORD>
supervised systemd
sudo systemctl restart redis-server

```
#### ПРОВЕРКА
```
redis-cli
127.0.0.1:6379> ping
127.0.0.1:6379> auth q1w2e3
OK
127.0.0.1:6379> ping
PONG
```
#### НАСТРОЙКА LARAVEL
```
composer require predis/predis
```
> [!NOTE]
> ФАЙЛ .env
```
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=<PASSWORD>
REDIS_PORT=6379
```
```
php artisan cache:clear
```
```
php artisan config:clear
```
```
php artisan queue:failed
```

## НАСТРОЙКА PORT FORWARD С HOST В WSL
> [!NOTE]
> ДЕЙСТВИЯ ДОЛЖНЫ ВЫПОЛНЯТЬСЯ В POWERSHELL

#### ADD FORWARD
```
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=<PORT> connectaddress=<ADDRESS_IN_WSL> connectport=<PORT_IN_WSL>
```
#### SHOW FORWARDS
```
netsh interface portproxy show all
```
#### DELETE FORWARDS
```
netsh interface portproxy delete v4tov4 listenport=<LISTEN_PORT> listenaddress=0.0.0.0
```

# ANDROID ПРИЛОЖЕНИЕ С VUE.JS
##### СОЗДАНИЕ VUE ПРИЛОЖЕНИЯ
```
npm create vue@latest <APP_FOLDER>
```
```
cd /<APP_FOLDER>
```
```
npm install
```
#### УСТАНОВКА Capacitor(СБОРЩИК ANDRPID APK)
```
npm install @capacitor/core @capacitor/cli
```
```
npx cap init
```
##### ДОБАВЛЕНИЕ ПЛАТФОРМЫ ANDROID
```
npx cap add android
```
#### СБОРКА ПРИЛОЖЕНИЯ ПОСЛЕ РАЗРАБОТКИ
```
npm run build
```
```
npm install @capacitor/android
```

##### СИНХРОНИЗАЦИЯ
```
npx cap sync android
# 8. (После изменений во Vue)
npm run build
npx cap sync android
# 9. Сборка APK (через Android Studio → Build)
# Build > Generate Signed App Bundle or APK
# Console build
cd android
./gradlew assembleDebug

adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## ЕСЛИ НЕТ ПОДСКАЗКИ В ОТДЕЛЬНОМ ФАЙЛЕ routes.js

```
/** @var \App\Models\User $user **/
/** @type {import('vue-router').RouteRecordRaw[]} */
const routes = [
  {
    path: '/',
    name: 'home',
    component: () => import('@/pages/Home.vue'),
  },
]
```

## НАСТРОЙКИ TELEGRAM WEBHOOK
#### SET WEBHOOK
```
https://api.telegram.org/bot<BOT_TOKEN>/setwebhook?url=<URL>
```
#### DELETE WEBHOOK
```
https://api.telegram.org/bot<BOT_TOKEN>/deleteWebhook?drop_pending_updates=true
```
#### GETINFO WEBHOOK
```
https://api.telegram.org/bot<BOT_TOKEN>/getWebhookinfo
```

