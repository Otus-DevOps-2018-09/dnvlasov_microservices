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
```
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
 


### ДЗ №15
#### Работа с сетью  в Docker
Создадим bridge-сеть в docker
```bash
docker network create reddit --driver bridge
```
Запустим наш проект reddit с использованием bridge-сети.
```bash
docker run -d --network=reddit mongo:latest
docker run -d --network=reddit dockerhub-login/post:1.0
docker run -d --network=reddit dockerhub-login/comment:1.0
docker run -d --network-reddit -p 9292:9292 dockerhub-login/ui:1.0
```
При просмотре  web по ip:9292 будет возникать ошибка 
"Can't show blog posts, some problems with the post service. Refresh?"
Для решения необходимо присвоение имени контейнерам
--name <name> (можно задать только 1 имя)
--network-alias <alias-name> (можно задать множество алиасов)
Остановим старые копии контейнеров
```bash
docker kill $(docker ps -q)
```
Запустим новые.
```
docker run -d --network=reddit --network-alias=post_db --network-alias=coment_db mongo:latest
docker run -d --network=reddit --network-alias=post login/post:1.0
docker run -d --network=reddit --network-alias=comment login/comment:1.0
docker rnn -d --network=reddit -p 9292:9292 login/ui:1.0
```
Проверим что предупреждение об ошибке пропала.

#### Запуск проекта в 2-х bridge сетях.
  
Запустим проект в 2-х bridge сетях. Так, чтобы сервис ui не имел доступа к базе данных
в соответствии со схемой.

 front_net             back_net
   ui        comment      db
               post 


Остановим старые копии контейнеров
```bash 
docker kill $(docker ps -q)
```
Создадим docker-сети
```bash
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
Запустим контейнеры
```bash
docker run --network=front_net -p 9292:9292 --name ui login/ui:1.0
docker run --network=back_net --name comment login/comment:1.0
docker run --network=back_net --name post login/post:1.0
docker run --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest
```
Контейнеры post и comment нужно поместить в обе сети
командой docker network connect подключим ко второй сети
```bash
docker network connect front_net post
docker network connect front_net comment
```
Зайдем на web по ip_address:9292 проверим что все работает без ошибок.
#### Сетевой стек схемы.
Заходим по ssh на docker-host и установим пакет bridge-utils  
```bash
docker-machine ssh docker-host
sudo apt-get update && sudo apt-get install bridge-utils
```
Выполним
```bash
docker network ls
```
Находим ID сетей созданных в рамках проекта.
```bash
ifconfig | grep br
br-30d790cce3d1 Link encap:Ethernet  HWaddr 02:42:f8:73:78:61
br-870ad93a7408 Link encap:Ethernet  HWaddr 02:42:cb:6b:ad:9f
br-dc9ad54119fe Link encap:Ethernet  HWaddr 02:42:df:f0:85:38

brctl show br-30d790cce3d1
bridge name     bridge id               STP enabled     interfaces
br-30d790cce3d1         8000.0242f8737861       no              veth4c9d2ae
                                                        veth8815442
                                                        vethd0e7d04

brctl show br-870ad93a7408
bridge name     bridge id               STP enabled     interfaces
br-870ad93a7408         8000.0242cb6bad9f       no              vetha8392b3
                                                        vethaa4417a
                                                        vethffde6e7

bridge name     bridge id               STP enabled     interfaces
br-dc9ad54119fe         8000.0242dff08538       no
```
Цепочка iptables  POSTROUTING (policy ACCEPT)
```bash
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  10.0.1.0/24          0.0.0.0/0
MASQUERADE  all  --  10.0.2.0/24          0.0.0.0/0
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  all  --  172.18.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  10.0.1.2             10.0.1.2             tcp dpt:9292
```
Правило отвечает за выпуск во внешнюю сеть контейнеров из bridge-сетей

Цепочка Docker и правила в ней отвечают за перенаправление трафика на адреса
конкретных контейнеров.
```bash
Chain DOCKER 
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:9292 to:10.0.1.2:9292
```
Процесс docker-proxy слушает на tcp-порту 9292
```bash
ps ax | grep docker-proxy
13479 ?        Sl     0:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 9292 -container-ip 10.0.1.2 -container-port 9292
```

#### Docker compose

В директории src создаем файл docker-compose.yml
```yml

version: '3.3'
services:
  post_db:
    container_name: post_db     
    image: mongo:3.2
    volumes:
      - post_db:/data/db
    networks:
      - reddit
  ui:
    build: ./ui
    container_name: ui
    image: ${USERNAME}/ui:1.0
    ports: 
      - 9292:9292/tcp
    networks:
      - reddit
  post:
    build: ./post-py
    container_name: post-py
    image: ${USERNAME}/post:1.0
    networks:
     - reddit 
  comment:
    build: ./comment
    container_name: comment
    image: ${USERNAME}/comment:1.0
    networks:
      - reddit
volumes:
  post_db:
networks:
  reddit:

```
Остановим контейнеры 
```bash
docker kill $(docker ps -q)
```
Экспортируем переменную USERNAME и запустим docker-compose
```
export USERNAME=<your-login>
docker-compose up -d
docker-compose ps
```bash

    Name                  Command             State           Ports         
----------------------------------------------------------------------------
src_comment_1   puma                          Up                            
src_post_1      python3 post_app.py           Up                            
src_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp             
src_ui_1        puma                          Up      0.0.0.0:9292->9292/tcp
```
Заходим на http://ip_docker_machine:9292 работает корректно.

- Изменеие docker-compose под кейс с множеством сетей, сетевых алисов.
- Параметризация с помощью переменных окружений:
  - порт публикации сервиса ui
  - версия сервисов
  - Параметризованные параметры запишите в отдельный файл c расширением .env
- Сущности создаваемые docker-compose можно изменить параметром containers_name

```yml
version: '3.3'
services:
  post_db:
    container_name: post_db     
    image: "mongo:${MONGO}"
    volumes:
      - post_db:/data/db
    networks:
      - back_net
  ui:
    build: ./ui
    container_name: ui
    image: "${USERNAME}/ui:${UI}"
    ports: 
      - ${NETWORK}
    networks:
      - front_net
  post:
    build: ./post-py
    container_name: post-py
    image: "${USERNAME}/post:${POST}"
    networks:
      - back_net
      - front_net  
  comment:
    build: ./comment
    container_name: comment
    image: "${USERNAME}/comment:${COMMENT}"
    networks:
      - back_net
      - front_net
volumes:
  post_db:
networks:
  front_net:
  back_net:
```   

### ДЗ №16
Создаем виртуальную машину.
```
#!/bin/bash
export GOOGLE_PROJECT=docker-224713
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-disk-size 100 \
--google-zone europe-west1-b \
gitlab-ci-2
```
Разрешаем подключение по HTTP/HTTPS
```
#!/bin/bash
gcloud compute firewall-rules create gitlab-ci\
 --allow tcp:80,tcp:443 \
 --target-tags=docker-machine \
 --description="gitlab-ci connections http & https" \
 --direction=INGRESS
```
Ставим docker с помощью ansible

ansible.cfg
``` 
[defaults]
inventory = ./inventory
remote_user = docker-user
private_key_file = ~/.docker/machine/machines/gitlab-ci/id_rsa
host_key_checking = False
retry_files_enabled = False
```
inventory
```
docker ansible_host=35.195.228.47
```
docker-composer-install
```yml
   
---
- name: install docker
  hosts: docker2
  become: true
  
  tasks:
    
  - name: install dependencies
    apt:
      name: 
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg2
      - software-properties-common
      - python-pip
    tags:
      - docker      
        
  
  - name: Debian add docker key
    apt_key:
      url: https://download.docker.com/linux/ubuntu/gpg
      state: present
    tags:
      - docker

  - name: Verify key
    apt_key:
       id: 0EBFCD88
       state: present
    tags:
      - docker
  
  - name: Add repository
    apt_repository:
      repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable    
      state: present
      update_cache: yes
    tags:
      - docker


  - name: install docker
    apt: 
     name: 
       - docker-ce
       - docker-compose
     update_cache: yes
    tags:
      - docker    
```
На сервере создаем необходимые директории
```bash
# mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
# cd /srv/gitlab/
# touch docker-compose.yml
```
Создаем файл docker-compose.yml
```
image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://<YOUR-VM-IP>'
  ports:
    - '80:80'
    - '443:443'
    - '2222:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'`
```
Запускаем docker-compose up -d

Проверям что все прошло успешно http://ip_gitlab

Указываем пароль root, переходим в админку выключаем регистацию новых пользователей.

Создаем private группу homework,создадим проект.

Добавим в username_microservices
```
git checkout -b gitlab-ci-1
git remote add gitlab http://<your-vm-ip>/homework/example.git
git push gitlab gitlab-ci-1
```
Переходим к определению CI/CD Pipeline для проекта, 
добавим в рерозиторий файл .gitlab-ci.yml
```yml
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo 'Building'

test_unit_job:
  stage: test
  script:
    - echo 'Testing 1'

test_integration_job:
  stage: test
  script:
    - echo 'Testing 2'

deploy_job:
  stage: deploy
  script:
    - echo 'Deploy'
```
Сохраняем файл.
```bash
git add .gitlab-ci.yml
git commit -m 'add pipeline definition'
git push gitlab gitlab-ci-1
```
Теперь если перейти в раздел CI/CD мы увидим, что пайплайн готов к
запуску.
Но находится в статусе pending / stuck так как у нас нет runner, запустим Runner и зарегистрируем его в интерактивном
режиме.

#### Runner
Перед тем, как запускать и регистрировать runner
нужно получить токен, переходим
```http
http://your_ip/homework/example/settings/ci_cd
```
Открываем  Runner, копируем 
- 3 Use the following registration token during setup:
На сервере, где работает Gitlab CI выполняем команду: 
```bash
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest 
```
После запуска Runner нужно зарегистрировать, это можно сделать командой
```bash
root@gitlab-ci:~# docker exec -it gitlab-runner gitlab-runner register
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://<YOUR-VM-IP>/
Please enter the gitlab-ci token for this runner:
<TOKEN>
Please enter the gitlab-ci description for this runner:
[38689f5588fe]: my-runner
Please enter the gitlab-ci tags for this runner (comma separated):
linux,xenial,ubuntu,docker
Whether to run untagged builds [true/false]:
[false]: true
Whether to lock the Runner to current project [true/false]:
[true]: false
Please enter the executor:
docker
Please enter the default Docker image (e.g. ruby:2.1):
alpine:latest
Runner registered successfully.
```
Если все получилось, то в настройках новый runner
- 4. Start the Runner!  
- Runners activated for this project
После добавления Runner пайплайн запустился

#### Добавим тестирование приложения reddit в pipeline
```
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m “Add reddit app”
git push gitlab gitlab-ci-1
```
#### Тестируем reddit

Изменим описание пайплайна в .gitlab-ci.yml
```yml
image: ruby:2.4.2
stages:
...
variables:
 DATABASE_URL: 'mongodb://mongo/user_posts'
before_script:
 - cd reddit
 - bundle install
...
test_unit_job:
 stage: test
 services:
 - mongo:latest
 script:
 - ruby simpletest.rb
```
#### Приложение reddit
В описании pipeline мы добавили вызов теста в файле simpletest.rb,
нужно создать его в папке reddit.
```
require_relative './app'
require 'test/unit'
require 'rack/test'

set :environment, :test

class MyAppTest < Test::Unit::TestCase
  include Rack::Test::Methods

  def app
    Sinatra::Application
  end

  def test_get_request
    get '/'
    assert last_response.ok?
  end
end
```

#### Приложение reddit
добавить библиотеку для тестирования в reddit/Gemfile приложения.
- gem 'rack-test'
```

source 'https://rubygems.org'

gem 'sinatra', '~> 2.0.1'
gem 'haml'
gem 'bson_ext'
gem 'bcrypt'
gem 'puma'
gem 'mongo'
gem 'json'
gem 'rack-test'

group :development do
    gem 'capistrano',         require: false
    gem 'capistrano-rvm',     require: false
    gem 'capistrano-bundler', require: false
    gem 'capistrano3-puma',   require: false
end
```
Теперь на каждое изменение в коде приложения будет запущен тест.

### ДЗ №17

Создадаем новый проект, example2.

Добавляем новый remote 
```bash
git checkout -b gitlab-ci-2
git remote add gitlab2 http://ip_gitlab-ci/homework/example2.git
git push gitlab2 gitlab-ci-2
```
Чтобы заработал pipeline включаем runner
1. Переименуем deploy stage в rewiew.
2. deploy_job меняем на deploy_dev_job
3. Добавляем environment
```yml
deploy_dev_job:
  stage: review
  script:
    - echo 'Deploy'
  environment:
     name: dev
     url: http://dev.example.com
```
После завершения pipeline с определением окружения переходим
`Environments` появится определение первого окружения `dev`

Определим два новых этапа: stage и production, первый будет
содержать job имитирующий выкатку на staging окружение, второй
на production окружение.
Определим эти job таким образом, чтобы они запускались с кнопки
```yml
stages:
  - build
  - test
  - review
  - stage
  - production
...
deploy_dev_job:
  stage: review
  script:
    - echo 'Deploy'
  environment:
     name: dev
    url: http://dev.example.com
staging:
  stage: stage
  when: manual
  script:
    - echo 'Deploy'
  environment:
    name: stage
    url: https://beta.example.com
production:
  stage: production
  when: manual
  script:
    - echo 'Deploy'
  environment:
    name: production
    url: https://example.com
```   
when: manual – говорит о том, что job должен быть
запущен по кнопке из UI.

На странице окружений  появиться оружения staging и production.

На production окружение выводится приложение с явно зафиксированной версией
(например, 2.4.10). 

Добавим в описание pipeline директиву, которая не позволит нам выкатить на staging и production код,
не помеченный с помощью тэга в git.
```yml
stages:
  - build
  - test
  - review
  - stage
  - production
… … …
stage:
  stage: stage
  when: manual
  only:
    - /^\d+\.\d+\.\d+/
  script:
    - echo 'Deploy'
  environment:
    name: stage
    url: https://beta.example.com
production:
  stage: production
  when: manual
  only:
    - /^\d+\.\d+\.\d+/
  script:
    - echo 'Deploy'
  environment:
    name: production
    url: https://example.com
```
Директива only описывает список условий, которые
должны быть истинны, чтобы job мог запуститься.
Регулярное выражение слева означает, что должен стоять
semver тэг в git, например, 2.4.10

Изменение, помеченное тэгом в git запустит полный pipeline.
```bash
git commit -a -m ‘#4 add logout button to profile page’
git tag 2.4.10
git push gitlab2 gitlab-ci-2 --tags
```
Динамические окружения
```yml
stages:
  - build
  - test
  - review
  - stage
  - production
. . .
branch review:
  stage: review
  script: echo "Deploy to $CI_ENVIRONMENT_SLUG"
  environment:
     name: branch/$CI_COMMIT_REF_NAME
     url: http://$CI_ENVIRONMENT_SLUG.example.com
  only:
     - branches
  except:
     - master
staging:
  stage: stage
  when: manual
```
Этот job определяет динамическое окружение
для каждой ветки в репозитории, кроме ветки
master

Теперь, на каждую ветку в git отличную от master
Gitlab CI будет определять новое окружени

### ДЗ №18

#### Подготовка окружения

 Создадим правило фаервола для Prometheus и Puma:
```bash
$ gcloud compute firewall-rules create prometheus-default --allow tcp:9090
$ gcloud compute firewall-rules create puma-default --allow tcp:9292
```
Содаем Docker хост в GCE настраиваем локальное окружение на работу с ним.
```bash
#!/bin/bash
export GOOGLE_PROJECT=docker-224713
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-disk-size 100 \
docker-host
```
Подключаемся к созданной docker-machine
```bash
$ eval $(docker-machine env docker-host)
```
Запускаем  Prometheus внутри Docker контейнера готовый образ с  DockerHub.
```bash
$ docker run -rm -p 9090:9090 -d --name prometheus prom/prometheus:v2.1.0
```
Открываем веб интерфейс.

По умолчанию сервер слушает на порту 9090, а IP адрес созданной VM
можно узнать, используя команду: 
```bash
$ docker-machine ip docker-host
```
Вкладка Console, которая сейчас активирована, выводит численное значение
выражений. Вкладка Graph, левее от нее, строит график изменений значений
метрик со временем.
Если кликнем по "insert metric at cursor", то увидим, что
Prometheus уже собирает какие-то метрики. По умолчанию он
собирает статистику о своей работе. Выберем, например,
метрику prometheus_build_info и нажмем Execute, чтобы
посмотреть информацию о версии.

#### Поясним результат вывода

prometheus_build_info{branch="HEAD",goversion="go1.9.1",instanc
e="localhost:9090", job="prometheus", revision=
"3a7c51ab70fc7615cd318204d3aa7c078b7c5b20",version="1.8.1"} 1 

`prometheus_build_info` - идентификатор собранной информации. 

`branch,goversion,instance,job,revision,version` - добавляет метаданных метрике, уточняет ее.
Использование лейблов дает нам возможность не ограничиваться
лишь одним названием метрик для идентификации получаемой
информации. Лейблы содержаться в {} скобках и представлены
наборами "ключ=значение".

В конце единица  - численное значение метрики, либо NaN, если
значение недоступно

#### Targets

Targets (цели) - представляют собой системы или процессы, за
которыми следит Prometheus. Помним, что Prometheus является
pull системой, поэтому он постоянно делает HTTP запросы на
имеющиеся у него адреса (endpoints). Посмотрим текущий список
целей.

#### Создание Docker образа

Создаем директорию monitoring/prometheus, в этой директории 
создем Dockerfile
```Dockerfile
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
```

#### Конфигурация

Определим простой конфигурационный файл
для сбора метрик с наших микросервисов. В
директории monitoring/prometheus создайте файл
prometheus.yml со следующим содержимым

```yml

---
global:
  scrape_interval: '5s'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets:
        - 'localhost:9090'
  - job_name: 'ui'
    static_configs:
      - targets:
        - 'ui:9292'
  - job_name: 'comment'
    static_configs:
      - targets:
        - 'comment:9292'
```
#### Создаем образ

```bash
$ export USER_NAME=username
$ docker build -t $USER_NAME/prometheus .
```
#### Образы микросервисов

Сборку образов теперь необходимо производить
при помощи скриптов docker_build.sh, которые есть
в директории каждого сервиса

#### Соберем images

Выполните сборку образов при помощи скриптов docker_build.sh 
```bash
for i in ui post-py comment; do cd src/$i; bash
docker_build.sh; cd -; done
```
Определите в docker/docker-compose.yml файле новый сервис. 
```

version: '3.3'
services:
  post_db:
    container_name: post_db     
    image: "mongo:${MONGO}"
    volumes:
      - post_db:/data/db
    networks:
      - back_net
  ui:
    container_name: ui
    image: "${USERNAME}/ui:${UI}"
    ports: 
      - ${NETWORK}
    networks:
      - front_net
  post:
    container_name: post-py
    image: "${USERNAME}/post:${POST}"
    networks:
      - back_net
      - front_net  
  comment:
    container_name: comment
    image: "${USERNAME}/comment:${COMMENT}"
    networks:
      - back_net
      - front_net
  prometheus:
    image: ${USERNAME}/prometheus
    ports:
      - '9090:9090'
    volumes:
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml' 
      - '--storage.tsdb.path=/prometheus' - Передаем доп. параметры в командной строке
      - '--storage.tsdb.retention=1d' - Задаем время хранения метрик в 1 день
    networks:
      - back_net
      - front_net      

            
volumes:
  prometheus_data:      
  post_db:
networks:
  front_net:
  back_net:        
```
Сборка Docker образов с данного момента производится через скрипт
`docker_build.sh` поэтому из файла `docker_compose.yml` удалениы все дерективы `build` и используется `image`

#### Запуск микросервисов
Поднимем сервисы, определенные в docker/dockercompose.yml 
```bash
$ docker-compose up -d 
```
#### Мониторинг состояния микросервисов

Посмотрим список endpoint-ов, с которых собирает
информацию Prometheus.

Endpoint-ы  в состоянии UP.

#### Healthchecks

Healthcheck-и представляют собой проверки того, что
наш сервис здоров и работает в ожидаемом режиме. В
нашем случае healthcheck выполняется внутри кода
микросервиса и выполняет проверку того, что все
сервисы, от которых зависит его работа, ему доступны.

Если требуемые для его работы сервисы здоровы, то
healthcheck проверка возвращает status = 1, что
соответсвует тому, что сам сервис здоров.

Если один из нужных ему сервисов нездоров или
недоступен, то проверка вернет status = 0. 

#### Состояние сервиса UI
В веб интерфейсе Prometheus выполним поиск по
названию метрики ui_health.

Построим график того, как менялось
значение метрики ui_health со временем.

#### Остановим post сервис
Останавливаем  сервис post на некоторое
время и проверим, как изменится статус ui сервиса,
который зависим от post. 
```bash 
$ docker-compose stop post
```
Метрика изменила свое значение на 0, что означает, что UI
сервис стал нездоров

Наберем в строке выражений ui_health_ и Prometheus нам предложит
дополнить названия метрик. 
```
ui_health_comment_availability
или
ui_health_post_availability
```
Выберем `ui_health_comment_availability`
видим, что сервис свой статус не менял в данный промежуток
времени

А с `ui_health_post_availability` все плохо.
 Поднимем post сервис. 
```bash
$ docker-compose start post 
```
Post сервис поправился.
UI сервис тоже.

#### Node exporter

Воспользуемся Node экспортер для сбора
информации о работе Docker хоста (виртуалки, где у
нас запущены контейнеры) и предоставлению этой
информации в Prometheus. 

Определим еще один сервис в docker/docker-compose.yml файле. 

```yml

services:
   ........
  node-exporter:
    image: prom/node-exporter:v0.15.2
    user: root
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points="^/(sys|proc|dev|host|etc)($$|/)"'      
    networks:
      - back_net

```
Чтобы  Prometheus следить за еще одним сервисом, 
добавим информацию о нем в конфиг
```yml

scrape_configs:
    ...

  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100'      


```
Соберем новый Docker для Prometheus:
```bash
monitoring/prometheus $ docker build -t $USER_NAME/prometheus .
```
Пересоздадим наши сервисы
```bash
$ docker-compose down
$ docker-compose up -d 
```

 
В списоке endpoint-ов Prometheus  появится еще один endpoint 

Получим информацию об использовании CPU 
```prometheus
node_load1
```
#### Проверим мониторинг

- Зайдем на хост: docker-machine ssh docker-host
- Добавим нагрузки: yes > /dev/null

Нагрузка выросла,мониторинг отображает повышение загруженности CPU

Ссылки на dockerhub 
```html
https://cloud.docker.com/u/verty/repository/docker/verty/prometheus
https://cloud.docker.com/repository/docker/verty/post
https://cloud.docker.com/repository/docker/verty/comment
https://cloud.docker.com/repository/docker/verty/ui
```


### ДЗ №19
#### Мониторинг приложения и инфраструктуры
Содаем Docker хост в GCE.
```bash

#!/bin/bash
export GOOGLE_PROJECT=docker-<number project>
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-disk-size 100 \
docker-host
```
Настраиваем локальное окружение
```bash
$ eval $(docker-machine env docker-host) 
```
Узнаем IP адрес
```bash
$ docker-machine ip docker-host
```
Разделим файлы Docker compose

Описание приложений в docker-compose.yml

Мониторинг  dockercompose-monitoring.yml

Настраиваем cAdvisor для наблюдения за состоянием наших Docker контейнеров.

cAdvisor будем запускать в контейнере. Добавим новый
сервис в наш компоуз файл мониторинга docker-compose-monitoring.yml
```yml
version: '3.3'
services:
  ... 
  cadvisor:
    image: google/cadvisor:v0.29.0
    volumes:
      - '/:/rootfs:ro'
      - '/var/run:/var/run:rw'
      - '/sys:/sys:ro'
      - '/var/lib/docker/:/var/lib/docker:ro'
    ports:
      - '8080:8080' 
    networks: 
      - front_net
```
Добавим информацию о новом сервисе в конфигурацию Prometheus, чтобы он начал собирать метрики

```yml
---
global:
  scrape_interval: '5s'

scrape_configs:

  - job_name: 'cadvisor'
    static_configs:
       - targets:
         - 'cadvisor:8080' 
```
Пересоберем образ Prometheus с обновленной конфигурацией.
```bash
$ export USER_NAME=dockerhab
$ docker build -t $USER_NAME/prometheus .
```
Запустим сервисы: 
```bash
$ docker-compose up -d
$ docker-compose -f docker-compose-monitoring.yml up -d 
```
Откроем страницу Web UI по адресу http://<docker-machine-host-ip>:8080

#### Grafana

Используем инструмент Grafana для визуализации данных из Prometheus.

Добавим новый сервис в docker-compose-monitoring.yml и сервис grafana в одну сеть с Prometheus.
```yml
#docker-compose-monitoring.yml 

services:
   ...
   grafana:
    image: grafana/grafana:5.0.0
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    networks:
      - front_net 
 volumes:
  grafana_data:      
```
Запустим новый сервис

```bash
$ docker-compose -f docker-compose-monitoring.yml up -d grafana 
```
Откроем страницу Web UI графаны по адресу 
http://<docker-mahine-host-ip>:3000 и используем для входа
логин и пароль админ пользователя, которые мы передали
через переменные окружения

После входа добавим источник данных

Зададим нужный тип и параметры подключения

Добавим dashboard загрузить json и поместим его в `monitoring/grafana/dashboard` поменяв 
название файла `DockerMonitoring.json`

Откроем вновь веб интерфейс Grafana и выберем импортировать шаблон

Загружаем скачанный дашборд. При загрузке указыываем источник данных для визуализации.

Появился набор графиков с информацией о состоянии хостовой системы и работе контейнеров.

#### Сбор метрик приложения

В качестве примера метрик приложения в сервис UI i мы добавили:
• счетчик ui_request_count, который считает каждый приходящий HTTP запрос (добавляя через
лейблы такую информацию как HTTP метод, путь, код возврата, мы уточняем данную метрику)
• гистограмму ui_request_latency_seconds, которая позволяет отслеживать информацию о времени
обработки каждого запроса

В качестве примера метрик приложения в сервис Post мы добавили:
• Гистограмму post_read_db_seconds, которая позволяет отследить информацию о времени
требуемом для поиска поста в БД

Добавим информацию о post сервисе в конфигурацию Prometheus, чтобы он начал собирать метрики и с него. 
```yml

---
global:
  scrape_interval: '5s'

scrape_configs:
 ...
  - job_name: 'post'
    static_configs:
         - targets:
           - 'post:5000'

```
Пересоберем образ Prometheus с обновленной конфигурацией.
```bash
$ export USER_NAME=username
$ docker build -t $USER_NAME/prometheus .
```
Пересоздадим нашу Docker инфраструктуру мониторинга:
```bash
$ docker-compose -f docker-compose-monitoring.yml down
$ docker-compose -f docker-compose-monitoring.yml up -d 
```
Добавим несколько постов в приложении и несколько
коментов, чтобы собрать значения метрик приложения.


Создание дашборда в Grafana.
В WEB UI по адресу ip_address:3000

Построим графики собираемых метрик приложения.
Выберем создать новый дашборд

Выбираем построить график

Жмем один раз на имя графика, затем выбираем Edit

Построим для начала простой график изменения счетчика HTTP запросов по времени.
Выберем источник данных и в поле запроса введем название метрики.

Далее достаточно нажать мышкой на любое место UI, чтобы убрать курсов из поля запроса, и Grafana выполнит запрос и
построит график.

В правом верхнем углу мы можем уменьшить временной интервал, на котором строим график, и
настроить автообновление данных.

Изменим заголовок графика и описание

Построим график запросов, которые возвращают код ошибки на этом же дашборде.

Добавим еще один график на наш дашборд.

Переходим в режим правки графика.

В поле запросов запишем выражение для поиска всех http запросов,
у которых код возврата начинается либо с 4 либо с 5 (используем
регулярное выражения для поиска по лейблу). Будем использовать
функцию rate(), чтобы посмотреть не просто значение счетчика за
весь период наблюдения, но и скорость увеличения данной
величины за промежуток времени (возьмем, к примеру 1-минутный
интервал, чтобы график был хорошо видим) 

Для проверки правильности нашего запроса обратимся по несуществующему HTTP пути,

http://docker-machinei-ip:9292/nonexistent

Проверим график (временной промежуток делаем
меньше для лучшей видимости графика).

Grafana поддерживает версионирование дашбордов, именно поэтому при сохранении нам предлагалось ввести сообщение,
поясняющее изменения дашборда. 

Добавим  третий по счету график на ваш дашборд. В поле запроса введите следующее выражение для вычисления 95
процентиля времени ответа на запрос
```
histogram_quantile(0.95, sum(rate(ui_request_latency_seconds_bucket[5m])) by (le))
```

#Мониторинг бизнес логики
Построим график скорости роста значения счетчика за последний час, используя функцию rate().
Это позволит нам получать информацию об активности пользователей приложения.
- Создадим новый дашборд, назовем его Business_Logic_Monitoring и построим график
функции rate(post_count[1h]) и rate(comment_count[1h])

Делаем экспорт дашборда и сохраняем его в директории monitoring/grafana/dashboards под
названием Business_Logic_Monitoring.json

#### Alertmanager

Создаем  новую директорию monitoring/alertmanager. В этой директории  Dockerfile
со следующим содержимым:
```Dockerfile
FROM prom/alertmanager:v0.14.0
ADD config.yml /etc/alertmanager/
```
В директории monitoring/alertmanager создаем файл config.yml, в
котором определим отправку нотификаций в тестовый слак
канал, через Incomig Webhook
```yml

global:
  slack_api_url: 'https://hooks.slack.com/services/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'
route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#dmitrii_vlasov'
```
1. Соберем образ alertmanager:
monitoring/alertmanager $ docker build -t $USER_NAME/alertmanager .
2. Добавим новый сервис в компоуз файл мониторинга.
```yml

version: '3.3'
services:
...
    alertmanager:
    image: ${USER_NAME}/alertmanager
    command:
      - '--config.file=/etc/alertmanager/config.yml'
    ports:
      - '9093:9093'
    networks:
      - front_net
```
Создадим файл alerts.yml в директории prometheus
```yml
monitoring/prometheus/alerts.yml

groups:
  - name: alert.rules
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute'
        summary: 'Instance {{ $labels.instance }} down'
```
Добавим операцию копирования данного файла в Dockerfile:
```Dockerfile
monitoring/prometheus/Dockerfile

FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
ADD alerts.yml /etc/prometheus/
```
Добавим информацию о правилах, в конфиг Prometheus
```yml

---
global:
  scrape_interval: '5s'

rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "alertmanager:9093"
```
Пересоберем образ Prometheus: 
```bash
$ docker build -t $USER_NAME/prometheus .
```
#### Проверка алерта
Пересоздадим нашу Docker инфраструктуру мониторинга: 
```bash
$ docker-compose -f docker-compose-monitoring.yml down
$ docker-compose -f docker-compose-monitoring.yml up -d 
```
Алерты можно посмотреть в веб интерфейсе Prometheus вкладка "Alerts"

#### Проверка алерта

Остановим один из сервисов и подождем одну минуту
```bash
$ docker-compose stop post 
```
В канал должно придти сообщение
```
AlertManager
 | [FIRING:1] InstanceDown(post:5000 post)
```
Ссылки на dockerhub
```http
https://cloud.docker.com/u/verty/repository/docker/verty/prometheus
https://cloud.docker.com/repository/docker/verty/post
https://cloud.docker.com/repository/docker/verty/comment
https://cloud.docker.com/repository/docker/verty/ui
https://cloud.docker.com/repository/docker/verty/alertmanager
```
### ДЗ №20

Выполните сборку образов при помощи скриптов docker_build.sh из корня репозитория.
```bash
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
Создадим Docker хост в GCE и настроим локальное окружение на работу с ним
```bash
#!/bin/bash
export GOOGLE_PROJECT=docker-224713
docker-machine create --driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-disk-size 100 \
--google-open-port 5601/tcp \
--google-open-port 9292/tcp \
--google-open-port 9411/tcp \
logging
```
Создадим отдельный compose-файл для нашей системы логирования в папке docker/

docker/docker-compose-logging.yml
```yml

version: '3'
services:
  fluentd:
    image: ${USERNAME}/fluentd
    ports:
      - "24224:24224"
      - "24224:24224/udp"
  elasticsearch:
    image: elasticsearch:5.6.14-alpine
    expose:
      - 9200
    ports:
      - "9200:9200"
  kibana:
    image: kibana:5.6.14
    ports:
```
Создадим образ Fluentd с нужной нам конфигурацией.

Создаём в вашем проекте microservices директорию logging/fluentd

В созданной директорий, создем Dockerfile.
```Dockerfile
FROM fluent/fluentd:v0.12
RUN gem install fluent-plugin-elasticsearch --no-rdoc --no-ri --version 1.9.5
RUN gem install fluent-plugin-grok-parser --no-rdoc --no-ri --version 1.0.0
ADD fluent.conf /fluentd/etc
```
В этой же  директории создаем файл конфигурации.
```conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
```
Соберем  docker image для fluentd
```bash
docker build -t $USER_NAME/fluentd .
```
Запускаем сервисы приложения.
```bash
docker/ $ docker-compose up -d
```
Выполним команду для просмотра логов post сервиса: 
```bash
docker/ $ docker-compose logs -f post 
```

#### Отправка логов во 	Fluentd
Определим драйвер для логирования для сервиса post внутри compose-файла 

```yml

version: '3.3'
services:
 ...
 post:
    container_name: reddit_post_1
    image: "${USERNAME}/post:${POST}"
    environment:
      - POST_DATABASE_HOST=post_db
      - POST_DATABASE=posts
      - ZIPKIN_ENABLED=${ZIPKIN_ENABLED} 
    depends_on:
      - post_db
    ports:
      - "5000:5000"
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: service.post
```
#### Сбор логов Post сервиса
Поднимем инфраструктуру централизованной системы
 
```bash
$ docker-compose -f docker-compose-logging.yml up -d
$ docker-compose down
$ docker-compose up -d 
```
Создадим несколько постов в приложении

Добавим фильтр для парсинга json логов, приходящих от post
сервиса, в конфиг fluentd
```conf

<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<filter service.post>
 @type parser
 format json
 key_name log
</filter> 

<filter service.ui>
  @type parser
  key_name log
  format grok
  grok_pattern %{RUBY_LOGGER}
</filter>

<filter service.ui>
  @type parser
  format grok
  grok_pattern service=%{WORD:service} \| event=%{WORD:event} \| request_id=%{GREEDYDATA:request_id} \| message='%{GREEDYDATA:message}'
  key_name message
  reserve_data true
</filter>

<filter service.ui>
  @type parser
  format grok
  key_name message
  grok_pattern service=%{WORD:service} \| event=%{WORD:event} \| path=%{URIPATH:path} \| request_id=%{UUID:request_id} \| remote_addr=%{IP:remote_addr} \| method=%{GREEDYDATA:method} \| response_status=%{INT:response_status} 
</filter>


<match *.**>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>
  <store>
    @type stdout
  </store>
</match>
 ```
Обновим образ 
```
docker build -t $USER_NAME/fluentd .
```
Перзапустим  docker-compose 
```bash
$ docker-compose -f docker-compose-logging.yml down
$ docker-compose -f docker-compose-logging.yml up -d
```

#### Zipkin
Добавим в compose-файл для сервисов логирования сервис распределенного трейсинга Zipkin
```yml
version: '3'

services:
  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9411:9411"

  fluentd:
    build: ./fluentd
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: elasticsearch
    expose:
      - 9200
    ports:
      - "9200:9200"

  kibana:
    image: kibana
    ports:
      - "8080:5601"
```
В docker/docker-compose.yml добавим для каждого сервиса поддержку ENV переменных и задаем
параметризованный параметр ZIPKIN_ENABLED
```
environment:
 - ZIPKIN_ENABLED=${ZIPKIN_ENABLED} 
```
В .env файле
```bash
ZIPKIN_ENABLED=true 
``` 
Обновим приложения
```bash
docker-compose up -d
```
Zipkin должен быть в одной сети с приложениями

```yml

version: '3'
services:
  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9411:9411"   
    networks:
      - back_net
      - front_net    
etworks:
  back_net:
  front_net: 
```
Пересоздадим наши сервисы
```bash
$ docker-compose -f docker-compose-logging.yml -f docker-compose.yml down
$ docker-compose -f docker-compose-logging.yml -f docker-compose.yml up -d
```

Откроем Zipkin WEB UI на порту 9411
       
### ДЗ №21

Пройдена Kubernetes The Hard Way.

kubectl apply -f проходит по созданным deployment-ам (ui, post, mongo, comment) и поды запускаются.

### ДЗ №22

####  Kubernetes. Запуск кластера и приложения. Модель безопасности.

Устанавливаем virtualbox minikube.

Запускаем  Minukube-кластер. 
```
$ minikube start

Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```
Minikube-кластер развернут. При этом автоматически был настроен конфиг kubectl. 

Проверим, что это так: 
```bash
$ kubectl get nodes 

NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   21h   v1.10.0
```
#### Deployment

Создаем компоненты.

##### ui-deployment.yml
```yml

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: ui
  template:
    metadata:
      name: ui-pod
      labels:
        app: reddit
        component: ui
    spec:
      containers:
      - image: verty/ui
        name: ui
```
Запустим в Minikube ui-компоненту
```bash
$ kubectl apply -f ui-deployment.yml 

deployment "ui" created        
```
Убедимся, что во 2,3,4 и 5 столбцах стоит число 3 (число реплик ui):

$ kubectl get deployment
```bash
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ui        3         3         3            3           23h
```
Пробросим порт на локальную машину с помощью kubectl
```bash
$ kubectl port-forward <pod-name> 8080:9292
```
Зайдем в браузере на http://localhost:8080 убеддимся что UI работает
и подключим остальные компоненты.

##### comment-deployment.yml
```yml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: comment
  labels:
    app: reddit
    component: comment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: comment
  template:
    metadata:
     name: comment
     labels:
       app: reddit
       component: comment
    spec:
      containers:
      - image: verty/comment
        name: comment
```
##### post-deployment.yml
```yml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: post
  labels:
    app: reddit
    component: post
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit
      component: post
  template:
    metadata:
      name: post 
      labels:
        app: reddit
        component: post
    spec:
      containers:
      - image: verty/post
        name: post
```
##### mongo-deployment.yml
```yml

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
    post-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
     name: mongo
     labels:
       app: reddit
       component: mongo
       comment-db: "true"
       post-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
```
Создаем  для связи объект `Service` - абстракция, которая определяет набор POD-ов (Endpoints) и
способ доступа к ним.

##### comment-service.yml 
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: comment
  labels:
    app: reddit
    component: comment
spec:
  ports:
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: comment
```
##### ui-service.yml
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: NodePort      
  ports:  
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
##### post-service.yml
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: post
  labels:
    app: reddit
    component: post
spec:
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: reddit
    component: post
```
ПО label-ам должны были быть найдены соответствующие
POD-ы. 
```bash
$ kubectl describe service comment | grep -i end

Endpoints:         172.17.0.13:9292,172.17.0.15:9292,172.17.0.16:9292
```
Изнутри любого POD-а должно разрешаться nslookup:
```bash
nslookup: can't resolve '(null)': Name does not resolve

Name:      comment
Address 1: 10.100.11.98 comment.default.svc.cluster.local
```
##### mongodb-service.yml
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
     app: reddit
     component: mongo
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo

```
Проверяем пробрасываем порт на ui pod
```bash
kubectl port-forward ui  9292:9292 
```
Заходим на http://locahost:9292

Не доступен blog post
```http
Can't show blog posts,some problems with the post service. Refresh& 
```
Решаем проблемму с помощью сервисов. Делаем service для comment и post.

##### comment-mongodb-service.yml
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: comment-db
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo
    comment-db: "true"
```
##### post-mongodb-service.yml
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: post-db
  labels:
    app: reddit
    component: mongo
    post-db: "true"
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: reddit
    component: mongo
    post-db: "true"
```
##### mongo-deployment.yml
```yml

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
  labels:
    app: reddit
    component: mongo
    comment-db: "true"
    post-db: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: mongo
  template:
    metadata:
     name: mongo
     labels:
       app: reddit
       component: mongo
       comment-db: "true"
       post-db: "true"
    spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongo-persistent-storage
        emptyDir: {}
```
##### Доступ к ui-сервису снаружи.

В ui-service.yml  добавляем новый тип service NodePort.
```yml

---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: NodePort      
  ports:  
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
Проверка
```bash
$ minikube service ui

|-------------|----------------------|-----------------------------|
|  NAMESPACE  |         NAME         |             URL             |
|-------------|----------------------|-----------------------------|
| default     | comment              | No node port                |
| default     | comment-db           | No node port                |
| default     | kubernetes           | No node port                |
| default     | mongodb              | No node port                |
| default     | post                 | No node port                |
| default     | post-db              | No node port                |
| default     | ui                   | http://192.168.99.128:31400 |
| kube-system | kube-dns             | No node port                |
|-------------|----------------------|-----------------------------|
```
#### Отделим среду для разработки приложения от всего остального кластера, создадим Namespace dev. 
##### dev-namespace.yml
```yml

---
apiVersion: v1
kind: Namespace
metadata:
  name: dev

$ kubectl apply -f dev-namespace.yml
```

Запускаем приложение в namespace.
```bash
$ kubectl apply -n dev -f kubernates/reddit
```
Проверяем  что запустилось, смотрим результат.
```bash
$ minikube service  ui -n dev
```
```http

Opening kubernetes service dev/ui in default browser...
   [1]Microservices Reddit in dev ui-5769658b67-hzglw container

  Menu

     * [2]All posts
     * [3]New post
```
Добавим инфу об окружении внутрь контейнера UI
```yml

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: ui
 ...
   spec:
      containers:
      - image: verty/ui
        name: ui
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace

```
#### Разворачиваем Kubernetes

В gcloud console, переходим в “kubernetes clusters”
Создаем кластер со следующими настройками.

- Версия кластера  - 1.8.10-gke.0
- Тип машины - небольшая машина (1,7 ГБ) (для экономии ресурсов)
- Размер - 2
- Базовая аутентификация - отключена
- Устаревшие права доступа - отключено
- Панель управления Kubernetes - отключено
- Размер загрузочного диска - 20 ГБ 

После создания  подключаемся к GKE для запуска  приложения,
убедимся что подключились к кластеру
```bash
kubectl config current-context

gke_docker-224713_europe-west1-b_standard-cluster-1
```
Запустим  приложение в GKE
Создадим dev namespace. 
```bash 
$ kubectl apply -f ./kubernetes/reddit/dev-namespace.yml 
```
Задеплоим все компоненты приложения в namespace dev.
```bash
$ kubectl apply -f ./kubernetes/reddit/ -n dev
```
Откроем Reddit в правила брандмауэра.

Найдем внешний ip-адрес любой ноды из кластера
```bash
$ kubectl get nodes -o wide 
 
NAME                                                STATUS   ROLES    AGE   VERSION          EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-standard-cluster-1-default-pool-2dd150ba-ljhm   Ready    <none>   4d    v1.10.11-gke.1   35.233.31.88    Container-Optimized OS from Google   4.14.65+         docker://17.3.2
gke-standard-cluster-1-default-pool-2dd150ba-qwwj   Ready    <none>   4d    v1.10.11-gke.1   35.195.171.80   Container-Optimized OS from Google   4.14.65+         docker://17.3.2
```
Порт публикации сервиса ui 
```bash
$ kubectl describe service ui -n dev | grep NodePort 

Type:                     NodePort
NodePort:                 <unset>  31689/TCP
```
Проверяем по адресу
```http
http://35.233.31.88:31689

   [1]Microservices Reddit in dev ui-557c59b75f-kkz5b container

  Menu

     * [2]All posts
     * [3]New post

```

### ДЗ №23

#### LoadBalancer
Настроим соответствующим образом Service UI 
```yml
ui-service.yml 

---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: LoadBalancer      
  ports:  
  - port: 80
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui

```
Применем конфигурацию
```bash
$ kubectl apply -f ui-service.yml -n dev
```
Смотрим внешний ip 
```bash
kubectl get service -n dev --selector component=ui

NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
ui     NodePort   10.7.242.81   33.135.3.22   80:30227/TCP     5d
```
#### Ingress

Создадим Ingress для сервиса UI

```yml
ui-ingress.yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
   name: ui
spec:
  backend:
      serviceName: ui
      servicePort: 9292
```
Применим конфиг
```bash 
$ kubectl apply -f ui-ingress.yml -n dev
```
Смотрим в сам кластер:
```bash 

NAME   HOSTS   ADDRESS          PORTS     AGE
ui     *       35.227.215.170     80       3h
```
Убираемодин балансировщик.
```yml
ui-service.yml

---
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: reddit
    component: ui
spec:
  type: NodePort      
  ports:  
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
```
Настроим на работу  Ingress Controller как классический веб
```yml
ui-ingress.yml

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ui
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: ui
          servicePort: 9292
```
##### Защитим наш сервис с помощью TLS. 
Смотрим external адрес Ingress
```bash
$ kubectl get ingress -n dev 

NAME   HOSTS   ADDRESS          PORTS     AGE
ui     *       35.227.215.170    80        3h
```
Подготовим сертификат используя IP как CN.
```bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=35.227.215.170"
```
Грузим сертификат в кластер kubernetes
```bash
$ kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev 
```
Проверяем  командой
```bash
$ kubectl describe secret ui-ingress -n dev


Name:         ui-ingress
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1111 bytes
tls.key:  1704 bytes
```

Настроим Ingress на прием только HTTPS траффика
```yml
ui-ingress.yml

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
   name: ui
   annotations:
           kubernetes.io/ingress.allow-http: "false"      
spec:
  tls:
  - secretName: ui-ingress  
  backend:
      serviceName: ui
      servicePort: 9292
```
Применим
```bash
$ kubectl apply -f ui-ingress.yml -n dev
```
Заходим на страницу  приложения по https

#### Network Policy

Найдите имя кластера
```bash
$ gcloud beta container clusters list

NAME                LOCATION        MASTER_VERSION  MASTER_IP      MACHINE_TYPE  NODE_VERSION   NUM_NODES  STATUS
standard-cluster-1  europe-west1-b  1.10.11-gke.1   35.205.148.14  g1-small      1.10.11-gke.1  2          RUNNING
```

Включим network-policy для GKE.

```bash
$ gcloud beta container clusters update standard-cluster-1 \
 --zone=europe-west1-b --update-addons=NetworkPolicy=ENABLED


$ gcloud beta container clusters update  standard-cluster-1 \
 --zone=europe-west1-b --enable-network-policy
```
Создадим mongo-network-policy
```

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-db-traffic
  labels:
    app: reddit
spec:
  podSelector:
    matchLabels:
      app: reddit
      component: mongo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
         app: reddit
         component: comment 
```
Применяем политику
```bash
$ kubectl apply -f mongo-network-policy.yml -n dev
```
Заходим в приложение.Post-сервис не может достучаться до базы

Добавляем Post-сервис в mongo-network-policy
```yml
 
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-db-traffic
  labels:
    app: reddit
spec:
  podSelector:
    matchLabels:
      app: reddit
      component: mongo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
         app: reddit
         component: comment 
    - podSelector:
        matchLabels:
         app: reddit
         component: post
```

#### Хранилище для базы
Создадим диск в Google Cloud
```bash
$ gcloud compute disks create --size=25GB --zone=europe-west1-b reddit-mongo-disk
```
Добавим новый Volume POD-у базы. 
```yml 
          
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
  labels:
 ...
     containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        gcePersistentDisk:
              pdName: reddit-mongo-disk
              fsType: ext4
```
Применем конфигурацию
```bash
$ kubectl apply -f mongo-deployment.yml
```
После пересоздания poda  приложение и добавим пост.  Удалим deployment.
```bash
$ kubectl delete deploy mongo -n dev
```
Снова создадим деплой mongo. 
```bash
$ kubectl apply -f mongo-deployment.yml -n dev
```
Проверим что пост остался на месте.

#### PersistentVolume
Создадим описание PersistentVolume 
```yml
mongo-volume.yml


---
apiVersion: v1
kind: PersistentVolume
metadata:
   name: reddit-mongo-disk
spec:
   capacity:
         storage: 25Gi
   accessModes:
         - ReadWriteOnce
   persistentVolumeReclaimPolicy: Retain
   gcePersistentDisk:
              fsType: "ext4" 
              pdName: "reddit-mongo-disk"
```
Добавим PersistentVolume в кластер
```bash
$ kubectl apply -f mongo-volume.yml -n dev  
```
#### PersistentVolumeClaim
Создадим описание PersistentVolumeClaim (PVC) 
```yml
mongo-claim.yml

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: mongo-pvc
spec:
   accessModes:
     - ReadWriteOnce
   resources:
     requests:
       storage: 15Gi
```
Добавим PersistentVolumeClaim в кластер
```bash
$ kubectl apply -f mongo-claim.yml -n dev
```
Подключим PVC к нашим Pod'ам
```yml

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
 ...
   spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        persistentVolumeClaim:
           claimName: mongo-pvc
```
Обновим описание нашего Deployment’а
```bash
$ kubectl apply -f mongo-deployment.yml -n dev      
```
Создадим описание StorageClass’а
```yml 
storage-fast.yml

---
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
   name: fast
provisioner: kubernetes.io/gce-pd
parameters:
     type: pd-ssd
```
Добавим StorageClass в кластер
```bash
$ kubectl apply -f storage-fast.yml -n dev
```
Создадим описание PersistentVolumeClaim 
```yml
mongo-claim-dynamic.yml

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
      requests:
          storage: 10Gi
```
Добавим StorageClass в кластер
```bash
$ kubectl apply -f mongo-claim-dynamic.yml -n dev
```
Подключим PVC к нашим Pod'ам 
```yml
mongo-deployment.yml 

---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: mongo
 ...
   spec:
      containers:
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        persistentVolumeClaim:
           claimName: mongo-pvc-dynamic      
```
Обновим описание нашего Deployment'а
```bash
$ kubectl apply -f mongo-deployment.yml -n dev
```
Посмотрит какие  у нас получились PersistentVolume'ы
```bash
$ kubectl get persistentvolume -n dev


NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                       STORAGECLASS   REASON   AGE
pvc-767eac0f-2462-11e9-a09e-42010a840134   10Gi       RWO            Delete           Bound       default/mongo-pvc-dynamic   fast                    1h
pvc-8ab285e7-2401-11e9-b2db-42010a84011e   15Gi       RWO            Delete           Bound       dev/mongo-pvc               standard                12h
reddit-mongo-disk                          25Gi       RWO            Retain           Available                                                       12h
```

