# xDebug + PHPStorm for macOS

1. Создаем alias своего локального IP. По сути единственное отличие от Windows/Linux. Там используется интерфейс docker0, не доступный в macOS. По этой же причине в настройки xDebug добавляется xdebug.remote_connect_back = off
```
sudo ifconfig lo0 alias 10.254.254.254
```
[Optional] Что бы не писать после каждой перезагрузки сохраняем .plist файл
```
sudo curl -o /Library/LaunchDaemons/com.yuklia.docker_localhost_alias.plist https://gist.github.com/yuklia/378a7350f8a2456e9513154b962c7cb0
```
2. Добавляем сервер в PHPStorm
   ![alt text](images/image_1.png?raw=true)
3. Настраиваем Debug порт. В моем случае 9000. xDebug будет стучаться на этот порт. Остальные галочки можно расставить на свой вкус.
   ![alt text](images/image_2.png?raw=true)
4. Конфигурируем DBG Proxy
   ![alt text](images/image_3.png?raw=true)
5. Пример Dockerfile на php:7-fpm с XDebug [В контейнере с ботом все это уже есть, а чего нет напишем в docker-compose.env, потому пропускаем]
```
FROM php:7-fpm

MAINTAINER yuklia

# upgrade the container
RUN apt-get update && \
    apt-get upgrade -y

RUN apt-get install -y --force-yes curl git nano zlib1g-dev \
    && docker-php-ext-install zip pdo pdo_mysql \
    && docker-php-ext-enable zip

# install composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer && \
    printf "\nPATH=\"~/.composer/vendor/bin:\$PATH\"\n" | tee -a ~/.bashrc

RUN pecl install xdebug-3.0.4 \
    && docker-php-ext-enable xdebug

ARG XDEBUG_INI=/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN echo "xdebug.mode=debug" >> ${XDEBUG_INI} \
    && echo "xdebug.client_host=10.254.254.254" >> ${XDEBUG_INI} \
    && echo "xdebug.client_port=9000" >> ${XDEBUG_INI} \
    && echo "xdebug.discover_client_host=false" >> ${XDEBUG_INI} \
    && echo "xdebug.idekey=PHPSTORM" >> ${XDEBUG_INI}

# install laravel envoy
RUN composer global require "laravel/envoy"

#install laravel installer
RUN composer global require "laravel/installer"

WORKDIR /var/www
```
6. В файле docker-compose.env для xDebug v3:
```
XDEBUG_CONFIG=xdebug.mode=debug;xdebug.client_host=10.254.254.254;xdebug.client_port=9000;xdebug.discover_client_host=false;xdebug.idekey=PHPSTORM;
```
7. docker-compose up -d --build app
8. Ставим brakepoint и слушаем входящие соединения
   ![alt text](images/image_4.png?raw=true)
   
### (Опционально) Debug HTTP Request

Можно просто дебажить блоки кода, где все равно на пейлоад, либо насоздавать пресетов пейлоада, добавив параметры запроса.
1. Настраиваем Docker сервер
   ![alt text](images/image_5.png?raw=true)
2. Добавляем конфигурацию с Docker сервером.
   ![alt text](images/image_6.png?raw=true)
3. Расставляем brakepoints и тыцяем на жучка
   ![alt text](images/image_7.png?raw=true)
