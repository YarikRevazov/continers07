# Лабораторная работа №7: Создание многоконтейнерного приложения

## Цель работы

Цель данной лабораторной работы — ознакомиться с принципами работы многоконтейнерного приложения с использованием `docker-compose`.

## Задание

Создать PHP-приложение, работающее на основе трёх контейнеров: `nginx`, `php-fpm` и `mariadb`, используя `docker-compose`.

## Выполнение работы

### 1. Подготовка проекта

Я создал репозиторий с названием `containers07` и склонировал его на свой компьютер.  
Внутри проекта я создал папку `mounts/site`, куда перенёс сайт, который разрабатывал в рамках предмета по PHP.

### 2. Игнорируемые файлы

В корне проекта я создал файл `.gitignore` со следующим содержанием:

```gitignore
# Ignore files and directories
mounts/site/*
```

### 3. Конфигурация nginx

В папке `nginx` я создал файл `default.conf` со следующим содержанием:

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index3.php;

    location / {
        try_files $uri $uri/ /index3.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index3.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 4. Docker Compose

Далее я создал файл `docker-compose.yml` в корне проекта:

```yaml
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
    env_file:
      - app.env

  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
      - app.env

  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

### 5. Файлы окружения

Я создал два файла с переменными окружения:

**mysql.env**
```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

**app.env**
```env
APP_VERSION=1.0.0
```

### 6. Запуск проекта

Для запуска контейнеров я использовал команду:

```bash
docker-compose up -d
```

После запуска я открыл [http://localhost](http://localhost) в браузере и проверил, что сайт отображается. Если сначала отображалась страница по умолчанию от nginx, я просто обновлял страницу — после полной загрузки всех контейнеров всё работало корректно.

## Ответы на вопросы

**1. В каком порядке запускаются контейнеры?**  
Контейнеры запускаются в том порядке, в каком они указаны в `docker-compose.yml`, но docker не гарантирует, что сервисы будут готовы к использованию сразу после запуска. В моём случае они запускались параллельно. Для управления зависимостями можно использовать `depends_on`, но я его здесь не использовал.

**2. Где хранятся данные базы данных?**  
Данные сохраняются в volume под названием `db_data`, который монтируется в контейнер базы данных по пути `/var/lib/mysql`.

**3. Как называются контейнеры проекта?**  
По умолчанию Docker формирует имена контейнеров как `<название_директории>_<имя_сервиса>_1`. У меня это были:
- `containers07_frontend_1`
- `containers07_backend_1`
- `containers07_database_1`

**4. Как добавить переменную окружения APP_VERSION для сервисов backend и frontend?**  
Я создал файл `app.env`, где указал:

```env
APP_VERSION=1.0.0
```

Затем я подключил его к обоим сервисам через параметр `env_file` в `docker-compose.yml`.

## Выводы

В ходе выполнения лабораторной работы я:

- Освоил базовые принципы работы с `docker-compose`.
- Настроил многоконтейнерное приложение с разделением по ролям: `frontend`, `backend`, `database`.
- Научился подключать конфигурационные файлы и использовать переменные окружения.
- Убедился в работоспособности связки `nginx` + `php-fpm` + `mariadb`.
