# docker_nginx

1. Для проверки этого задания следует установить пакеты:
docker ce
docker-compose
Если это уже было выполнено, достаточно перейти к следущему пункту. 
Описана установка Docker на Линукс Ubuntu -  версия подставляется по параметру $(lsb_release -cs)
Удаление старых версий (если они были)
$ sudo apt-get remove docker docker-engine docker.io
Обновление пакетов
$ sudo apt-get update
Установка пакетов приложений, необходимых для установки Docker CE
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
Добавление официального PGP ключа
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
Проверка
$ sudo apt-key fingerprint 0EBFCD88
Добавление адреса в список репозиториев
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
Обновление и установка
$ sudo apt-get update
$ sudo apt-get install docker-ce

2. Структура каталогов:

/opt/docker/ -- тут находится docker-compose.yml
/opt/docker/nginx -- тут находится Dockerfile
--- Dockerfile - кастомный nginx, который впоследствии загружен на DockerHub https://hub.docker.com/repository/docker/raibeartruadh/docker_nginx
--- Dokcerfile2 - простой вариант, когда я стягиваю мой кастомный образ из DockerHub 
/opt/docker/nginx/params -- тут находится конфиг nginx.conf
/opt/www - тут находится наша страничка

Содержимое доступно в репозитории: https://github.com/RaibeartRuadh/docker_nginx.git

3. Создание кастмного образа nginx
Был выбран вариант со сборкой nginx из исходников. Описание см. в файле /opt/docker/nginx/Dockerfile 
В качестве OS был выбран Linux alpine:3.11
В качестве версии nginx - стабильная 1.19.3 с сайта https://nginx.org/
Общий процесс выглядит следующим образом:
Я стягиваю образ alpine:3.11 на котором произвожу:
а. предварительную установку пакетов для компиляции nginx
б. стягиваю, распаковываю и перехожу в джиректорию с исходником nginx
в. выполняю конфигурирование через команду ./configure с параметрами, которые мне нужны.
г. выполняю make и make install
д. добавляю пользователя nginx
e. подчищаю за собой каталоги (чистота везде)
ж. добавляю наш файл конфигурации, который лежит в каталоге params
и. прописываваю expose 
к. подаю команду запуска nginx в контейнере 

в файле docker-compose я прописываю параметры:

...services:
....nginx:
........build: ./nginx # - откуда я делаю сборку (там находится Dockerfile)
........container_name: nginx
........ports:
............- "80:80"
........network_mode: host
........volumes:
............- /var/log/nginx:/var/log/nginx
............- /opt/www/:/opt/www

Далее. Сборку выполняем из каталога /opt/docker
$ cd /opt/docker
$ sudo docker-compose build nginx
$ sudo docker-compose up -d nginx

проверяем, что контейнер запущен
$ sudo docker ps
Проверяем нашу страничку (она будет доступна на localhost) Через команду curl или можно в браузере
$ curl http://localhost

Образ рабочий. Теперь его заливаем на DockerHub
Выполняем:
$ docker login
$ docker tag docker_nginx raibeartruadh/docker_nginx
потом "пушим" его
$ docker push raibeartruadh/docker_nginx

Теперь его можно выкачать командой
$ docker pull raibeartruadh/docker_nginx:latest

Останавливаем контейнер с nginx
$ sudo docker stop nginx
Удаляем контейнер
$ sudo docker rm nginx
Удаляем образ (ID можно узнать через команду sudo docker ps)
$ sudo docker rmi aed41e21a306

Меняем в директории /opt/docker/nginx название Dockerfile2 на Dockerfile
Содержимое файла указывает, что я скачиваю мой собранный образ, при сборке я добавляю свой файл конфигурации.

FROM raibeartruadh/docker_nginx:latest
WORKDIR /opt/docker/nginx/
ADD params/nginx.conf /etc/nginx/nginx.conf

Выполняем сборку из каталога /opt/docker
$ cd /opt/docker
$ sudo docker-compose build nginx
$ sudo docker-compose up -d nginx

проверяем, что контейнер запущен
$ sudo docker ps
Проверяем нашу страничку (она будет доступна на localhost) Через команду curl или можно в браузере
$ curl http://localhost

# Определите разницу между контейнером и образом
Образ docker это шаблон, который используется при сборке контейнера. При этом команды в Dockefile, написанные построчно выполняют функцию слоев. Для оптимизации образа важно писать ёмкие инструкции. Мы может добавлять определенные параметры при этом, наслаивая образ. С контейнером так поступить не получится. 

# Можно ли в контейнере собрать ядро?
Можно. Есть подобная работа на github  https://github.com/naftulikay/docker-ubuntu-kernel-build
Тем не менее, это лишь показывает возможности docker, использовать такое преимущество не получится, так как всё будет зависить от машины-хоста

# ссылки:
https://github.com/RaibeartRuadh/docker_nginx.git
https://hub.docker.com/repository/docker/raibeartruadh/docker_nginx




