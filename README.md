# Microservices

# Docker machine (GCE)

docker-machine create --driver google \
--google-project docker-185809 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host

# Open port in GCE FW :9292
gcloud compute firewall-rules create reddit-app --allow tcp:9292 --priority=65534 --target-tags=docker-machine --description="Allow TCP connections" --direction=INGRESS
