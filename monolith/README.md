# Microservices

# Docker machine (GCE)

docker-machine create --driver google \
--google-project docker-185820 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host

eval $(docker-machine env docker-host)
docker-machine env docker-host
docker-machine ssh docker-host

# Open port in GCE FW :9292
gcloud compute firewall-rules create reddit-app --allow tcp:9292 --priority=65534 --target-tags=docker-machine --description="Allow TCP connections" --direction=INGRESS

# Docker build
docker build -t reddit:latest .

# Startup container
docker run --name reddit -d --network=host reddit:latest

# Push image to hub.docker.com
docker tag reddit:latest shevchenkoav/otus-reddit:1.0

docker push shevchenkoav/otus-reddit:1.0

# Delete instance
gcloud compute instances delete docker-host --zone=europe-west1-b

# Delete dangling images
docker system prune

docker rmi $(docker images -f "dangling=true" -q)
