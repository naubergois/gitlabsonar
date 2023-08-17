# Use Docker Swarm 

## Pre-requisite

### on the master
```shell
sudo apt-get update  # update repo ref
sudo apt install -y python3.8-venv docker.io python3-pip  # install docker ce package 
sudo usermod -aG docker $USER
```
### on the runner 
```shell
sudo apt-get update  # update repo ref
sudo apt install -y python3.8-venv docker.io python3-pip  # install docker ce package 
sudo usermod -aG docker $USER
```
## log out and login again 
Check with a ```docker ps``` on master and runner  


## Check on master  
```shell
docker network ls
brctl show
ip -4 addr show dev docker0
```
## Swarm init on master 
```shell
docker swarm init   # init the swarm 
docker node ls    # Check 
docker node update --label-add gitlab=master <HOSTNAME> # set a label
docker node inspect --pretty <HOSTNAME> | grep -A 5 Labels: # check
docker swarm join-token worker # get the token again 
```
## On the runner 
docker swarm join --token 8ixxxxxxx xxxxxx:2377  # add a runner node

##  On the master 
```shell
sudo mkdir -p /opt/gitlab/config
sudo mkdir -p /opt/gitlab/logs
sudo mkdir -p /opt/gitlab/data
```

## Run docker-compose for swarm 
```yaml
 docker stack deploy -c docker-compose.yml gitlab
 watch docker service ls 
```

## Register the runner  
```shell
docker exec -it --user root  gitlab_gitlab-runner.xxxxx /bin/bash

gitlab-runner register --name <Project name> \
--url http://<ip_master_address> \
--registration-token <token>
```

## reload gitlab runner
```shell
# uncheck shared runner
gitlab-runner verifiy
#start manullaly the runner in docker exec  
gitlab-runner run 
```


