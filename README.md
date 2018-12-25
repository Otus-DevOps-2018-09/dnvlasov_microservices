# dnvlasov_microservices
dnvlasov microservices repository

###  Введение в docker
- Устанвливаем packages repository to allow apt use a repository ower HTTP

```bash
$ sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common
```
Добавим ключ GPG
```bash
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
Проверим что ключи добавились а apt-key
```bash
$ sudo apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22
	Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```
Добавим в source.list stable repository
```bash
$ sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/debian \
  $(lsb_release -cs) \
  stable"
```
Делаем обновление пакетов
```bash
sudo apt update
```
И устанавливаем docker-ce.
```bash
$ sudo apt install docker-ce=17.06.2~ce-0~debian
```

Запустим первый контейнер.
```bash
docker run hello-world
```
Список запущенных контейнеров
```
$ sudo docker ps

CONTAINER ID  IMAGE   COMMAND    CREATED   STATUS    PORTS       NAMES
```
Список всех контейнеров
```
$ docker ps 

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
8ea689d42d82        ubuntu:16.04        "/bin/bash"         4 hours ago         Exited (1) 4 hours ago                         kind_merkle
7639619723b9        ubuntu:16.04        "/bin/bash"         4 hours ago         Exited (137) 3 hours ago                       kind_cohen
68c1fd6f7647        hello-world         "/hello"            4 hours ago         Exited (0) 4 hours ago                         jovial_kare
```
Список сохраненных образов
```
$ docker images 

REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
my-docker/ubuntu-tmp-file   latest              0ac4cc36d7e7        25 hours ago        116MB
ubuntu                      16.04               a51debf7e1eb        2 weeks ago         116MB
hello-world                 latest              4ab4c602aa5e        2 months ago        1.84kB
```
Docker run  каждый раз запускает новый контейнер. Если не указывать флаг --rm при запуске docker run,
то после остановки контейнер в месте с содержимым остается на диске.
```
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreateAt}}\t{{.Names}}"
 
CONTAINER ID        IMAGE               CREATED AT                      NAMES
8ea689d42d82        ubuntu:16.04        2018-12-04 14:04:03 +0300 MSK   kind_merkle
7639619723b9        ubuntu:16.04        2018-12-04 14:02:25 +0300 MSK   kind_cohen
68c1fd6f7647        hello-world         2018-12-04 13:59:12 +0300 MSK   jovial_kare
```
Запуск нового процесса внутри контейнера например bash
start  запускает  уже созданный контейнер
attach подсоединяет  терминал  к созданному контейнеру
```
docker start 8ea689d42d82

docker attach 8ea689d42d8
ENTER

docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8ea689d42d82        ubuntu:16.04        "/bin/bash"         2 days ago          Up 4 minutes                            kind_merkle

```
Запускаем новый процесс внутри контейнера

```
docker exec -it  8ea689d42d82 bash
```
Docker commit создает image из  контенера, контейнер при этом остается запущенным.
```

sha256:839f6a935c85d9ff8bbbe668fc872b908bafccf25abc1d64e297da58961b3551

docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
my-docker/ubuntu-tmp-file   latest              0ac4cc36d7e7        2 days ago          116MB
ubuntu                      16.04               a51debf7e1eb        2 weeks ago         116MB
hello-world                 latest              4ab4c602aa5e        2 months ago        1.84kB
````
### ДЗ №13

- Создаём  новый проект  в GCP с названем docker
- Создаем новый конфигурационный файл командой
```bash
gcloud init
```
Создаем аутентификационные данные для доступа к приложению
```bash
gcloud auth application-default login
```
Создаем скрипт для создания docker-machine 
```bash
#!/bin/bash
export GOOGLE_PROJECT=docker-XXXXXX
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host
```
После запускаем и проверяем что Docker успешно запустился
```bash
$bash docker.machine.up
$docker-machine ls

NAME          ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER     ERRORS
docker-host   -        google   Running   tcp://222.222.222.222:2376           v18.09.0   
```
И переключаемся на него
```bash
eval $(docker-machine env docker-host)
```
Для дальнейшей работы нужно создать следующие файлы
- Dockerfile - текстовое описание нашего образа
- mongod.conf - подготовленный конфиг для mongodb
- db_config - содержит переменную окружения со ссылкой на mongodb
- start.sh - скрипт запуска приложения
Вся работа происходят в папке docker-monolith
```vim
mongodb.conf

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
 
start.sh

#!/bin/bash

/usr/bin/mongod --fork --logpath /var/log/mongod.log --config /etc/mongodb.conf

source /reddit/db_config

cd /reddit && puma || exit

db_config

DATABASE_URL=127.0.0.1
```
Создадим образ приложения ubuntu 16.04
```vim

FROM ubuntu:16.04

RUN apt-get update
RUN apt-get install -y mongodb-server ruby-full ruby-dev build-essential git
RUN gem install bundler
RUN git clone -b monolith https://github.com/express42/reddit.git

COPY mongod.conf /etc/mongod.conf
COPY db_config /reddit/db_config
COPY start.sh /start.sh

RUN cd /reddit && bundle install
RUN chmod 0777 /start.sh

CMD ["/start.sh"]
```
Выполним сборку образа
```bash
$ docker build -t reddit:latest .

```
Теперь  запустим наш контейнер
```
$ docker run --name reddit -d --network=host reddit:latest
```
Проверим результат 
```
$ docker-machine ls

NAME        ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER     ERRORS
docker-host   -        google   Running   tcp://222.222.222.222:2376       v17.09.0-ce
```
Разрешим входящий TCP-трафик  на порт 9292 выполнив команду

``` bash
gcloud compute firewall-rules create reddit-app \
 --allow tcp:9292 \
 --target-tags=docker-machine \
 --description="Allow PUMA connections" \
 --direction=INGRESS
```
Загрузим образ на docker hub
```
docker login
docker tag reddit:latest login/otus-reddit:1.0 
docker push login/otus-reddit:1.0
```
И еще проверка
```json
docker inspect <your-login>/otus-reddit:1.0

[
    {
        "Id": "sha256:05b2cb5a01083e2de7693d151285d0b37f784f3d0143a9fca56c279944a64c91",
        "RepoTags": [
            "reddit:latest",
            "login/otus-reddit:1.0"
        ],
        "RepoDigests": [
            "login/otus-reddit@sha256:c867984622cda30f7d03bf0d13d7f431765861cf1b897e9ab53c7d5a444e7aed"
        ],
        "Parent": "sha256:ca8af415e9713a12f1c6ae21cabfcca690d5b9aacd89ec142678af893eac3642",
        "Comment": "",
        "Created": "2018-12-08T12:03:46.409836381Z",
        "Container": "3c9ee932cc71abbf89e760d5ed7a1474958490c05dbda6a8c795f565bef40cd6",
        "ContainerConfig": {
            "Hostname": "3c9ee932cc71",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/start.sh\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:ca8af415e9713a12f1c6ae21cabfcca690d5b9aacd89ec142678af893eac3642",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "18.09.0",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/start.sh"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:ca8af415e9713a12f1c6ae21cabfcca690d5b9aacd89ec142678af893eac3642",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 681077551,
        "VirtualSize": 681077551,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/4da0e10e48eafc9f44ac4165578c4a6d285b8e94c5de9a72f48c95a4d66a4b12/diff:/var/lib/docker/overlay2/0d78366f3c5bdee9d1847250e501d2893c6356397143672723714b3a7f1c20ba/diff:/var/lib/docker/overlay2/7de0c82195a605e8e4ab4f4cc92f1f54aad3914a07b85efce02195fec648029d/diff:/var/lib/docker/overlay2/bb7a269f4456258522a33d889eeacf873c5a59f12a1efd25137a09a33cdff22c/diff:/var/lib/docker/overlay2/e5e62bfbb09afc72daba59178ceda8e8bc6b6ba35c2bfa27ed3b559919908603/diff:/var/lib/docker/overlay2/402e1798479db11ae552517f0dde3bed2ebc1c9a9093027bb3bb160335d44935/diff:/var/lib/docker/overlay2/8d69fb56ca3872a0b72b767adc5202174c6c9a7a68220d7ddd8c85c02387f24f/diff:/var/lib/docker/overlay2/701695213c9aebe88f2f003b2678359175ca1108cff6494b939d6d06a4f72fde/diff:/var/lib/docker/overlay2/715ca389e78c8809c070e0d6af18e65e79e205171e5db0a3d0db0cf2f89030e9/diff:/var/lib/docker/overlay2/cb8e26649a1cbdf0ec4f83f19f0fdff7386eff4bd9b634df69676ee983e5760b/diff:/var/lib/docker/overlay2/c93817764554a5a98c739701be797d24c7a5bb20a122d1f79233a39ebcc5fd63/diff:/var/lib/docker/overlay2/5a2e1cb34bc0d3701c5ad95b50539f278916baf248cd9a691556c1734a535b53/diff",
                "MergedDir": "/var/lib/docker/overlay2/ead4f7b20e6815b14c7be7389358cb191041c64f7b16b4a7456fbfe9f874c3af/merged",
                "UpperDir": "/var/lib/docker/overlay2/ead4f7b20e6815b14c7be7389358cb191041c64f7b16b4a7456fbfe9f874c3af/diff",
                "WorkDir": "/var/lib/docker/overlay2/ead4f7b20e6815b14c7be7389358cb191041c64f7b16b4a7456fbfe9f874c3af/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:41c002c8a6fd36397892dc6dc36813aaa1be3298be4de93e4fe1f40b9c358d99",
                "sha256:647265b9d8bc572a858ab25a300c07c0567c9124390fd91935430bf947ee5c2a",
                "sha256:819a824caf709f224c414a56a2fa0240ea15797ee180e73abe4ad63d3806cae5",
                "sha256:3db5746c911ad8c3398a6b72aa30580b25b6edb130a148beed4d405d9c345a29",
                "sha256:9a4bc08cbc6aa6710d48343231853e47fc6e5fc16c6d901629d29540e6056a50",
                "sha256:6c2c6a379303cdb7a00f2175d163cd626fe537e6c77e1d23c2b6ac813a9e992b",
                "sha256:298d1d015927df78a17f6be9ce732e714096fb360f1ffec8e6677131d0b405c5",
                "sha256:2cea5f58ac5fa068a2deee03a22990abca4b54f595b05cecd1a7155913157f89",
                "sha256:e5532dac33bcaa71f178a2b4a5051630e5dd25d3df3de1c9a34f6914c7280010",
                "sha256:8825a868bcb118c35703eb3139a4b531e9f0dc83f0ae7e507d8838697c12c27c",
                "sha256:6efbada0efe2e0f262de308758bd58379e9ae85fdf555ca2fe5c6d2546680c37",
                "sha256:9900fb8186201e327dfc6dd08da85c419762377780a737748b664998d4659275",
                "sha256:fd6d299e2682fb2adae33b352f9a6f0c296aa323585a453063bb40fb8b4f2e59"
            ]
        },
        "Metadata": {
            "LastTagTime": "2018-12-08T17:24:56.840982562+03:00"
        }
    }
]
```
```bash
docker inspect <your-login>/otus-reddit:1.0 -f '{{.ContainerConfig.Cmd}}' 

[/bin/sh -c #(nop)  CMD ["/start.sh"]]
```

### ДЗ №14
#### Docker-образы Мкросервисы
Подключаемся к Docker host
```bash
NAME          ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER     ERRORS
docker-host     -        google   Running   tcp://35.240.74.230:2376           v18.09.0   
eval $(docker-machine env docker-host)
```
Скачаем zip архив и распакуем его содержимое внутрь  репозитория,
В нутри  репозитория у нас появиться каталог reddit-microervices переменуем его в `src`
Теперь наше приложение состоит из трех компонентов
- `post-py` - сервис отвечающий за написание постов
- `comment` - сервис отвечающий за написание коменрариев
- `ui`      - веб-интерфейс, работающий с другими сервисами

Создадим файл ./post-py/Dockerfile

```docker
FROM python:3.6.0-alpine

WORKDIR /app
ADD . /app

RUN pip install -r /app/requirements.txt

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

ENTRYPOINT ["python3", "post_app.py"]
```
./comment/Dockerfile
```docker

FROM ruby:2.2
RUN apt-get update -qq && apt-get install -y build-essential

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV COMMENT_DATABASE_HOST comment_db
ENV COMMENT_DATABASE comments

CMD ["puma"]
```
./ui/Dockerfile
```docker

FROM ruby:2.2 
RUN apt-get update -qq \
    && apt-get install -y  build-essential 

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
```
Скачаем последний образ MongoDB:
```bash
docker pull mongo:latest
```
Соберем  образы с нашими сервисами:
```bash
docker build -t login/post:1.0 ./post-py
docker build -t login/comment:1.0 ./comment
docker build -t login/ui:1.0 ./ui
```
Cозздадим специальную сеть для приложения
```
docker network create reddit
```
Запустим наши контейнеры
```bash
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:laest
docker run -d --network=reddit --network-alias=post login-dockerhub/post:1.0
docker run -d --network=reddit --network-alias=post login-dockerhub/comment:1.0
docker run -d --network=reddit -p 9292:9292 login-dockerhub/ui:1.0
```
Что было сделанно.
- Создана bridge-сеть для контейнеров
- Запустили контейнеры в этой сети
- Добавили сетевые алиасы контейнерам
#### Образы приложения
Образы преложения ранимают много места для одного приложения

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
verty/ui            1.0                 94ba693f3bcf        8 days ago          781MB
verty/comment       1.0                 a2f2b084ff87        8 days ago          773MB
verty/post          1.0                 f524ee9a94a2        8 days ago          102MB
mongo               latest              525bd2016729        4 weeks ago         383MB
ubuntu              16.04               a51debf7e1eb        4 weeks ago         116MB
ruby                2.2                 6c8e6f9667b2        7 months ago        715MB
python              3.6.0-alpine        cb178ebbf0f2        21 months ago       88.6MB
```
- Улучшаем  образ сервиса ui
поменяем содержимое ./ui/Dockerfile
```docker
FROM ubuntu:16.04 
RUN apt-get update  \
    && apt-get install -y ruby-full ruby-dev build-essential \
    && gem install bundler --no-ri --rdoc 

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
ADD Gemfile* $APP_HOME/
RUN bundle install
ADD . $APP_HOME

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
```
Пересоберем ui  `docker build -t login/ui:2.0 ./ui`

```

POSITORY          TAG                 IMAGE ID            CREATED             SIZE
ui                2.0                 f17bd1160df4        6 minutes ago       455MB
ui                1.0                 94ba693f3bcf        9 days ago          781MB
comment           1.0                 a2f2b084ff87        9 days ago          773MB
post-py           1.0                 f524ee9a94a2        9 days ago          102MB
mongo             latest              525bd2016729        4 weeks ago         383MB
ubuntu            16.04               a51debf7e1eb        4 weeks ago         116MB
ruby              2.2                 6c8e6f9667b2        7 months ago        715MB
python            3.6.0-alpine        cb178ebbf0f2        21 months ago       88.6MB
```

#### Перезапуск приложения
Выключим старые  копии контейнеров
```
docker kill $(docker ps -q)
```
Запустим новые копии контейнеров.
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post dockerhub-login/post:1.0
docker run -d --network=reddit --network-alias=comment dockerhub-login/comment:1.0
docker run -d --network=reddit -p 9292:9292 dockerhub-login/ui:2.0
```
Проверим на http://ip_docker_host:9292. Ранее созданный пост пропал 
вместе с остановкой контейнера mongo.
- Создадим Docker volume:
```
$ docker volume create reddit_db
```
И подключим его к контейнеру с MongoDB.

Выключим старые копии контейнеров:
```
docker kill $(docker ps -q)
```
Запустим новые копии контейнеров:
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post dockerhub-login/post:1.0
docker run -d --network=reddit --network-alias=comment dockerhub-login/comment:1.0
docker run -d --network=reddit -p 9292:9292 dockerhub-login/ui:2.0 
```
- Заходим на http://ip_docker_host:9292
- Пишем пост
- Перезапустим контейнеры bash docker-conteiner_restart
- Проверяем наличие поста
 
