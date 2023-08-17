# Gitlab course

## Pre-requisite
```shell
sudo apt-get update  # update repo ref
sudo apt install -y python3.8-venv docker.io python3-pip  # install docker ce package 
sudo usermod -aG docker $USER
```

## Install a virtualenv for docker-compose

```shell
cd 
git clone https://github.com/crunchy-devops/gitlab-course.git  # git clone your repo locally
cd gitlab-course  # change directory
python3 -m venv venv  # install a python virtualenv
source venv/bin/activate  # activate the virtual env
pip3 install pip --upgrade
pip3 install docker-compose  # install python package docker-compose 
```

### Run portainer for getting gitlab initial password
```shell
docker volume create portainer_data
docker run -d -p 32125:8000 -p 32126:9443 --name portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data portainer/portainer-ce:latest 
```
Get portainer tool up and running   
go to container gitlab-course_gitlab-ce_1 and select a exec_console icon   
enter cat /etc/gitlab/initial_root_password
or directly using docker exec  
`docker exec -it git cat /etc/gitlab/initial_root_password`
Here, it's randomly generated password available for 24 hours   
Copy and paste this password in your browser at     
http://<ip_address>  
if you have some trouble reset password ```sudo gitlab-rake "gitlab:password:reset"```  
Log in as root using this password

## Configure manually a Gitlab runner 
```shell
docker run --rm -it -v /opt/gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner register
```

## Change gitlab-runner configuration 
Go to `/etc/gitlab-runner`
add `network_mode="gitlab-course_prodnetwork"` at the end of `[runners.docker]` block code
do a `gitlab-runner restart`

## Install a gitlab-runner on dedicated host
```shell
sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"
sudo chmod +x /usr/local/bin/gitlab-runner
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
```

## Connect to Nexus Artifact repository 
```shell
 docker exec -i gitlab-course_nexus_1 cat /nexus-data/admin.password
 # Create a docker hosted repo 
 # in HTTP textbox enter 20000 
 # check allow anomynous docker pull 
 # Enable Docker v1 API
 # Deployment policy : disable redeploy
 # in main menu in the left hand Security section select Realms and 
 # assign  Docker Bearer Token Realm
docker network inspect gitlab-course_prodnetwork
sudo vi /etc/docker/daemon.json 
 # Add 
{
        "insecure-registries": ["nexus:20000"]
}
## Add in /etc/hosts ip_of_container_nexus nexus  
docker-compose up -d
docker login http://nexus:20000
```