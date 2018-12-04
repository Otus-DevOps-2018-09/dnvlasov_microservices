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
Список всех кнтейнеров
```
$ docker ps 

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
8ea689d42d82        ubuntu:16.04        "/bin/bash"         4 hours ago         Exited (1) 4 hours ago                         kind_merkle
7639619723b9        ubuntu:16.04        "/bin/bash"         4 hours ago         Exited (137) 3 hours ago                       kind_cohen
68c1fd6f7647        hello-world         "/hello"            4 hours ago         Exited (0) 4 hours ago                         jovial_kare
2730ef4b4ee5        hello-world         "/hello"            5 hours ago         Exited (0) 5 hours ago                         awesome_bassi
```
