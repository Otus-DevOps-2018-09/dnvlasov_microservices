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

