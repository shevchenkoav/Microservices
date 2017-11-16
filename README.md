# Microservices

## post-py

Сервис отвещающий за написание постов

## comment

Cервис отвечающий за написание комментариев

## ui

Веб-интерфейс для других сервисов

## DB

 - MongoDB

```bash
docker pull mongo:latest

docker build -t shevchenkoav/post:1.0 ./post-py

docker build -t shevchenkoav/comment:1.0 ./comment

docker build -t shevchenkoav/ui:1.0 ./ui
```

- or change image version for new version Dockerfile

```bash
docker build -t shevchenkoav/ui:2.0 -f ./ui/Dockerfile_2.0 ./ui/
docker build -t shevchenkoav/ui:3.0 -f ./ui/Dockerfile_3.0 ./ui/
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

### Create bridge-network with alt.aliases and new env for each containers.

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
- Stop running containers

```bash
docker kill $(docker ps -q)
```
