# Microservices
- Сервис отвещающий за написание постов
- Cервис отвечающий за написание комментариев
- Веб-интерфейс для других сервисов

## Docker machine (GCE) create
```bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host
```

```bash
eval $(docker-machine env docker-host)
docker-machine env docker-host
docker-machine ssh docker-host
```
## Open port in GCE FW :9292
```bash
gcloud compute firewall-rules create reddit-app --allow tcp:9292 --priority=65534 --target-tags=docker-machine --description="Allow TCP connections" --direction=INGRESS
```

## Build containers

```bash
docker pull mongo:latest

docker build -t shevchenkoav/post:1.0 ./post-py

docker build -t shevchenkoav/comment:1.0 ./comment

docker build -t shevchenkoav/ui:1.0 ./ui
```

- or change image ui version for new version Dockerfile

```bash
docker build -t shevchenkoav/ui:2.0 -f ./ui/Dockerfile_2.0 ./ui/
docker build -t shevchenkoav/ui:3.0 -f ./ui/Dockerfile_3.0 ./ui/
### without upgrade&&update
docker build -t shevchenkoav/ui:4.0 -f ./ui/Dockerfile_4.0 ./ui/
```

### Create bridge-network with aliases

```bash
docker network create reddit

docker run -d --network=reddit -v reddit_db:/data/db \
--network-alias=post_db --network-alias=comment_db mongo:latest

docker run -d --network=reddit \
--network-alias=post shevchenkoav/post:1.0

docker run -d --network=reddit \
--network-alias=comment shevchenkoav/comment:1.0

docker run -d --network=reddit \
-p 9292:9292 shevchenkoav/ui:3.0
```

#### Create bridge-network with alt.aliases and new env for each containers.

```bash
docker run -d --network=reddit -v reddit_db:/data/db \
--network-alias=post_db --network-alias=comment_db \
-e POST_SERVICE_HOST='post_reddit' \
-e COMMENT_SERVICE_HOST='comment_reddit' \
-e COMMENT_DATABASE_HOST='comment_db' \
mongo:latest

docker run -d --network=reddit \
--network-alias=post_reddit \
-e POST_SERVICE_HOST='post_reddit' \
-e COMMENT_SERVICE_HOST='comment_reddit' \
-e COMMENT_DATABASE_HOST='comment_db' \
shevchenkoav/post:1.0

docker run -d --network=reddit \
--network-alias=comment_reddit \
-e POST_SERVICE_HOST='post_reddit' \
-e COMMENT_SERVICE_HOST='comment_reddit' \
-e COMMENT_DATABASE_HOST='comment_db' \
shevchenkoav/comment:1.0

docker run -d --network=reddit \
-p 9292:9292 \
-e POST_SERVICE_HOST='post_reddit' \
-e COMMENT_SERVICE_HOST='comment_reddit' \
-e COMMENT_DATABASE_HOST='comment_db' \
shevchenkoav/ui:1.0
```

- Open url and test app.

## Stop & delete running containers/instances

```bash
docker kill $(docker ps -q)
```

### Push image to hub.docker.com
```bash
docker tag reddit:latest shevchenkoav/otus-reddit:1.0
```
```bash
docker push shevchenkoav/otus-reddit:1.0
```
### Delete dangling images
```bash
docker system prune
```
- or
```bash
docker rmi $(docker images -f "dangling=true" -q)
```
### Delete instance (GCE)
```bash
docker-machine rm docker-host
```
# HW 17

- docker-machine create

### for example: tests, export, import, build, parsing (interface loopback)
```bash
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
```
### for example: tests, export, import, build, parsing (interface loopback)

```bash
docker run --network host --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
```

```bash
docker-compose -p 'compose_project_name' up -d
```

#HW 19 GITLAB CI

## Info about docker drivers (GCE)
https://docs.docker.com/machine/drivers/

## GCE machine types
https://cloud.google.com/compute/docs/machine-types

## Docker machine (GCE) create
```bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type n1-standard-1 \
--google-disk-size 100 \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) gitlab-ci

or

docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type n1-standard-1 \
--google-disk-size 100 \
--google-open-port 80 \
--google-open-port 443 \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
gitlab-ci
```

```bash
## Config environment
eval $(docker-machine env gitlab-ci)
```

docker-machine ssh gitlab-ci

### Runner
docker run -d --name gitlab-runner --restart always \
   -v /srv/gitlab-runner/config:/etc/gitlab-runner \
   -v /var/run/docker.sock:/var/run/docker.sock \
   gitlab/gitlab-runner:latest

### Runner is need to be registered
docker exec -it gitlab-runner gitlab-runner register

### if need to restart gitlab ci
sudo gitlab-ctl restart
sudo gitlab-ctl status
sudo docker restart container_name

## HW 21
- Prometheus: запуск, конфигурация, знакомство с Web UI
- Мониторинг состояния микросервисов
- Сбор метрик хоста с использованием экспортера

### Create fw rule for prmt & puma
```bash
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
```

### create docker host & configure local env
```bash
docker-machine create --driver google \
--google-project infra-185820 \
--google-machine-image https://www.googleapis.com/compute/v1/projects/
ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
vm1
```
```bash
eval $(docker-machine env vm1)
```

Возможные ошибки в ходе выполнения работы:
1. Контейнеры собраны не со всеми зависимостями (post-requirement-post_app.py-import prometheus), будут падать сразу же в exited, нужно пересобрать.
2. CRLF -> LF
3. в HEAD стали попадать файлы вида new file: "comment/build_info.txt\r" единственный найденный пока способ как от этого избавляться git stash, при том что файл добавлен в игнор.


docker push shevchenkoav/ui
docker push shevchenkoav/comment
docker push shevchenkoav/post
docker push shevchenkoav/prometheus
