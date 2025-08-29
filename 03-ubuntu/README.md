# 🐧 Ubuntu Server: Гайд по управлению сервером для разработчика

## 📁 Работа с файлами и папками

### Создание и удаление
```bash
# Создать папку проекта
sudo mkdir -p /var/www/my-project
sudo mkdir -p /var/www/my-project/{public,logs,backups}

# Создать файл
sudo touch /var/www/my-project/index.html

# Рекурсивное удаление папки
sudo rm -rf /var/www/old-project

# Удаление файла
sudo rm /var/www/my-project/temp-file.txt
```

### Просмотр и навигация
```bash
# Просмотр содержимого с правами и владельцем
ls -la /var/www/

# Просмотр дерева каталогов
tree /var/www/ -L 2

# Поиск файлов
find /var/www/ -name "*.html" -type f
```

## 👥 Пользователи и права

### Просмотр пользователей
```bash
# Текущий пользователь
whoami

# Все пользователи
cat /etc/passwd

# Группы пользователя
groups www-data
id www-data
```

### Настройка прав chmod
```bash
# Дать права на чтение/запись/выполнение владельцу, чтение/выполнение группе и остальным
sudo chmod 755 /var/www/my-project

# Рекурсивно для всей папки
sudo chmod -R 755 /var/www/my-project

# Дать права на запись для группы
sudo chmod g+w /var/www/my-project/public
```

### Настройка владельца chown
```bash
# Сменить владельца на www-data
sudo chown www-data:www-data /var/www/my-project

# Рекурсивно сменить владельца и группу
sudo chown -R www-data:www-data /var/www/my-project

# Сменить только группу
sudo chgrp -R www-data /var/www/my-project
```

### Права для веб-сервера
```bash
# Стандартная настройка прав для веб-проекта
sudo chown -R $USER:www-data /var/www/my-project
sudo chmod -R 755 /var/www/my-project
sudo chmod -R g+w /var/www/my-project/public
```

## 🌐 Настройка веб-серверов

### Apache
```bash
# Создание виртуального хоста
sudo nano /etc/apache2/sites-available/my-project.conf
```

**Пример конфига:**
```apache
<VirtualHost *:80>
    ServerName my-project.test
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/my-project/public
    
    ErrorLog ${APACHE_LOG_DIR}/my-project_error.log
    CustomLog ${APACHE_LOG_DIR}/my-project_access.log combined
    
    <Directory /var/www/my-project/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
# Активация сайта
sudo a2ensite my-project.conf

# Проверка конфигурации
sudo apache2ctl configtest

# Перезагрузка Apache
sudo systemctl reload apache2
```

### Nginx
```bash
# Создание конфига сайта
sudo nano /etc/nginx/sites-available/my-project
```

**Пример конфига:**
```nginx
server {
    listen 80;
    server_name my-project.test;
    root /var/www/my-project/public;
    index index.html index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    access_log /var/log/nginx/my-project_access.log;
    error_log /var/log/nginx/my-project_error.log;
}
```

```bash
# Активация сайта
sudo ln -s /etc/nginx/sites-available/my-project /etc/nginx/sites-enabled/

# Проверка конфигурации
sudo nginx -t

# Перезагрузка Nginx
sudo systemctl reload nginx
```

## 🔧 Управление службами systemctl

```bash
# Просмотр статуса службы
sudo systemctl status apache2
sudo systemctl status nginx
sudo systemctl status php8.3-fpm

# Запуск/остановка/перезагрузка
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2

# Включение автозагрузки
sudo systemctl enable apache2
sudo systemctl disable apache2

# Просмотр всех служб
systemctl list-units --type=service
```

## 📡 Копирование файлов

### SCP (Secure Copy)
```bash
# Копирование с локальной машины на сервер
scp -r local-folder/ user@server_ip:/var/www/my-project/

# Копирование с сервера на локальную машину
scp user@server_ip:/var/www/my-project/file.txt ./local-folder/

# Копирование с указанием порта
scp -P 2222 file.txt user@server_ip:/var/www/
```

### RSYNC
```bash
# Синхронизация папок
rsync -avz --progress local-folder/ user@server_ip:/var/www/my-project/

# Синхронизация с удалением лишних файлов
rsync -avz --delete local-folder/ user@server_ip:/var/www/my-project/

# Синхронизация по SSH с исключениями
rsync -avz -e ssh --exclude='node_modules' --exclude='.git' ./ user@server_ip:/var/www/my-project/
```

## 🖥️ Работа с процессами

### Просмотр процессов ps
```bash
# Все процессы
ps aux

# Процессы конкретного пользователя
ps -u www-data

# Поиск процесса
ps aux | grep nginx
```

### Управление процессами kill
```bash
# Завершение процесса по PID
kill 1234

# Принудительное завершение
kill -9 1234

# Завершение всех процессов по имени
pkill nginx
killall nginx
```

## 📝 Работа с текстом (nano)

```bash
# Открытие файла
sudo nano /etc/nginx/nginx.conf

# Основные команды nano:
# Ctrl+O - Сохранить
# Ctrl+X - Выйти
# Ctrl+W - Поиск
# Ctrl+\ - Замена
# Ctrl+K - Вырезать строку
# Ctrl+U - Вставить
```

## 🌐 Работа с сетью (curl)

```bash
# Проверка доступности сайта
curl -I http://localhost

# Скачивание файла
curl -O https://example.com/file.zip

# Отправка POST запроса
curl -X POST -d "param=value" http://localhost/api

# С заголовками
curl -H "Content-Type: application/json" -X GET http://localhost/api

# Сохранить результат в файл
curl -o output.txt http://localhost
```

## 🔍 Полезные команды мониторинга

```bash
# Просмотр логов в реальном времени
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/apache2/error.log

# Мониторинг ресурсов
htop
top

# Свободное место на диске
df -h

# Размер папок
du -sh /var/www/*
```

## 🚀 Быстрые команды для повседневной работы

```bash
# Перезагрузка веб-серверов
alias restart-web="sudo systemctl restart nginx apache2 php8.3-fpm"

# Просмотр логов
alias log-nginx="sudo tail -f /var/log/nginx/error.log"
alias log-apache="sudo tail -f /var/log/apache2/error.log"

# Проверка конфигов
alias check-nginx="sudo nginx -t"
alias check-apache="sudo apache2ctl configtest"
```

Этот гайд покрывает основные команды для управления сервером Ubuntu в повседневной работе разработчика! 🎯
