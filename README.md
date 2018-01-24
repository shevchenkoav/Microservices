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
--google-project docker-185820 \
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

docker-compose logs --follow

docker push shevchenkoav/ui
docker push shevchenkoav/comment
docker push shevchenkoav/post
docker push shevchenkoav/prometheus


# HW23 Мониторинг приложения и инфраструктуры (app&infra monitoring)

- Docker containers monitoring
- Visualise metrics
- Collecting application metrics and business metrics
- Configure alerting

### Create docker host
```bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-open-port 80/tcp \
--google-open-port 3000/tcp \
--google-open-port 8080/tcp \
--google-open-port 9090/tcp \
--google-open-port 9292/tcp \
--google-open-port 9093/tcp \
vm1
```

### Configure local environment
```Bash
eval $(docker-machine env vm1)
```

## cAdvisor

- add new service in yml file

```bash
cadvisor:
  image: google/cadvisor:latest
  volumes:
    - '/:/rootfs:ro'
    - '/var/run:/var/run:rw'
    - '/sys:/sys:ro'
    - '/var/lib/docker/:/var/lib/docker:ro'
  ports:
    - ${CADVISOR_HOST_PORT}:${CADVISOR_CONTAINER_PORT}/tcp
  networks:
    - back_net
    - front_net
```
- update prometheus yml file

```bash
- job_name: 'cadvisor'
  static_configs:
      - targets:
        - 'cadvisor:8080'
```

```bash
Login Dockerhub
export USER_NAME=
docker build -t $USER_NAME/prometheus .
 ```

## Grafana
add new service in yml file

```Bash
  grafana:
    image: grafana/grafana
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    ports:
      - 3000:3000
  volumes:
    grafana_data:
```

docker-compose up -d grafana

## add info about post service in prometheus yml
```Bash
  - job_name: 'post'
    static_configs:
      - targets:
        - 'post:5000'
```

### App metrics
#### add dashboard (ui requests, http request with error status code, http response time)

### Business_Logic_Monitoring
#### add post_count & comment_count graphics

## Add alertmanager for prometheus
#### Add Dockerfile for alertmanager & config.yml
```Bash
FROM prom/alertmanager
ADD config.yml /etc/alertmanager/
```

#### config.yml (create incoming webhook 'https://devops-team-otus.slack.com/apps/A0F7XDUAZ-incoming-webhooks?page=1')
```Bash
global:
  slack_api_url: 'https://hooks.slack.com/services/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'

route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#artem-starostenko'
```

### build image
docker build -t $USER_NAME/alertmanager .

add alert.rules to /Prometheus
```Bash
# Alert for any instance that is unreachable for >5 minutes.
ALERT InstanceDown
  IF up == 0
  FOR 1m
  ANNOTATIONS {
    summary = "Instance {{ $labels.instance }} down",
    description = "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.",
  }
```

#### add rule to Dockerfile Prometheus
```bash
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
ADD alert.rules    /etc/prometheus/
```

#### update Prometheus config
```Bash
rule_files:
  - "alert.rules"
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "alertmanager:9093"
```

#### rebuild Dockerfile
docker build -t $USER_NAME/prometheus .

## Push images
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
docker push $USER_NAME/alertmanager

---------------
# Logging. HW 25.

## Create docker host & config local environment

### create host
docker-machine create --driver google \
--google-project docker-185820 \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
--google-open-port 5601/tcp \
--google-open-port 9292/tcp \
--google-open-port 9411/tcp \
logging

### Configure local environment
eval $(docker-machine env logging)

### make docker-compose-logging.yml
### create fluentd service ./fluentd
### make fluentd congif file
### add inforward & copy plagins
### add to post service logging driver to docker-compose.yml

```bash
ports:
    - "5000:5000"
logging:
  driver: "fluentd"
  options:
    fluentd-address: localhost:24224
    tag: service.post
```

```Bash
docker build -t $USER_NAME/fluentd .
docker build -t shevchenkoav/post:2_0 -f ./post-py/Dockerfile_2_0 ./post-py/
```

### pulling images
```bash
docker pull $USER_NAME/ui
docker pull $USER_NAME/comment
docker pull $USER_NAME/post:2_0 ### (for python2.7)
docker pull $USER_NAME/prometheus
docker pull $USER_NAME/alertmanager
docker pull $USER_NAME/fluentd
```

```Bash
docker-compose -f docker-compose-logging.yml up -d
docker-compose down
docker-compose up -d
```

### if need python 2.7
docker build -t shevchenkoav/post:2_0 -f ./post-py/Dockerfile_2_0 ./post-py/

docker-compose up -d

Выполняем команду для просмотра логов post сервиса:
docker-compose logs -f post

### validate docker-compose.yml
docker-compose -f docker-compose.yml config

25HW don't finished...

-----

# 26-27 HW. Swarm

- Build or pull UI services
- Create Machines:

```Bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list
--filter ubuntu-1604-lts --uri) \
master-1
```

```Bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list
worker-1
```

```Bash
docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list
--filter ubuntu-1604-lts --uri) \
worker-2
```

### add infrastr.
