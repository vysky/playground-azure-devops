# playground-azure-devops

## summary

1. create azure devops organization if havent, and push a git repo to azure repos

2. create a vm and install docker on it

3. start a sonarqube container and integrate it with azure devops

4. build a custom tomcat image, start it and integrate it with azure devops

5. paste the pipeline code and run the pipeline

---

## docker

### create a vm and install docker

1. go to azure portal and create a vm using ubuntu 20.04 server image

2. ssh into the vm using `ssh -i <private key> <username>@<ip address>`

3. install docker using the following commands

```
sudo apt-get update

sudo apt-get install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

---

## sonarqube

### install sonarqube extension

1. go to azure devops and click on the marketplace icon (top right hand corner)

2. search for sonarqube and install the extension

https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube

### pull and run sonarqube using docker

1. ssh into the vm and run `docker pull sonarqube:lts`

2. once the image is pulled, run `docker run -d --name "sonarqube" -p 9000:9000 sonarqube:lts`

3. if encounter any permission issue, run with `sudo`

### add inbound port rule

1. go to the vm > networking > inbound port rules, create a new rule and change the destination port ranges to 9000 (leave other values as default)

2. once the rule is added, sonarqube shoud be accessible through ip:9000

### integrate sonarqube and azure devops

1. go to azure devops and create a personal access token (save it for later use) with full access (should not for production use)

2. go to sonarqube > adminstration > alm integration > azure devops, paste the personal access token and url

3. go to projects > add project > azure devops, paste the personal token when prompted

4. select the project created > set up selected repository

5. select with azure pipelines when prompted and instructions on how to continue will appear

6. generate the token (step 2) when prompted (save it for later use), or go to adminstrative > security > users > token icon and generate from there

7. go back to azure devops > project settings (bottom left corner) > service connection > create service connection > sonarqube > next

8. follow the instructions in sonarqube and enter the url, token and name, and save the service connection

9. continue following the instructions in sonarqube to edit the pipeline, or use the azure-pipelines.yml file in the repo

### resources

https://www.youtube.com/watch?v=GfgUTj4rp7c

https://www.youtube.com/watch?v=ApMeWoRkv-A

https://docs.sonarqube.org/latest/analysis/azuredevops-integration/

### issues encountered

1. SonarQube server [http://localhost:9000] can not be reached

fixed it by adding `options: '-Dsonar.host.url=http://40.112.183.95:9000'` to the maven task

2. Not authorized. Analyzing this project requires authentication. Please provide a user token in sonar.login or other credentials in sonar.login and sonar.password.

fixed it by adding `options: -Dsonar.login=38bcac0a64b8f7672c6057836b1407bf708778e1`

the token is the one generated when following the instruction to integrate sonarqube and azure

add the token as env variable in actual project, see https://stackoverflow.com/a/63920566

or

go to sonarqube > adminstration > configuration > security (not the security beside configuration), and turn off force user authentication

3. The SonarQube Prepare Analysis Configuration must be added.

probably due to seperating the tasks into different stages, combining all the stage into single stage and the issue is resolved

---

## tomcat

### install apache tomcat deployment extension

1. go to azure devops and click on the marketplace icon (top right hand corner)

2. search for apache tomcat deployment and install the extension

https://marketplace.visualstudio.com/items?itemName=ms-vscs-rm.apachetomcat

### build and run tomcat using docker

tomcat
	use the dockerfile i created for jenkins
	go to the vm with docker, follow my guide to create custom tomcat image and run it
	remember to create an inbound rule for the vm
	source port * and destincation 8888 as i put 8888 when i start the tomcat custom image

docker build tomcat -t "tomcat-custom"

docker run -d --name "tomcat" -p 8888:8080 tomcat-custom

### add inbound port rule

1. go to the vm > networking > inbound port rules, create a new rule and change the destination port ranges to 8888 (leave other values as default)

2. once the rule is added, tomcat shoud be accessible through ip:8888

### deployment task

add tomcat deployment task to the pipeline, see azure-pipelines.yml file

### issues encountered

1. /usr/bin/curl failed with return code: 26

the working directory is purged once the stage is completed thus curl cannot open the file to deply it to tomcat

tried copying the war file to another directory (example tmp) to deploy in another stage, but still cannot as everything will still be purged (may be azure devops uses a fresh vm image)

need to use publish pipeline artifact task during the build stage and download pipeline artifact stage during the deploy stage

### resources

https://stackoverflow.com/a/52547885

http://aka.ms/tomcatdeploymenttask

http://go.microsoft.com/fwlink/?LinkId=550988

https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/pipeline-artifacts

---

## create agent (on hold)

1. create an agent using azure vm (cannot use wsl as wsl do not have access to systemctl) or vmware

2. install ubuntu 20.04 server

3. go to azure devops > pipelines > environment

4. create a new environment by selecting vm as the resource and linux as the os, then copy the registration script

5. login to the new ubuntu server and run the registration script

6. go to azure devops repos, copy the code snippet to push the code to azure devops repos

7. on your local git repo, paste in the code snippet to push it to azure devops repos

8. if password is required, go back to azure devops repos to generate the account info

9. once the code is pushed to azure devops repos, go to azure devops pipelines and create a new pipeline

10. as the code is in azure devops repos, select the azure devops repos option when creating

---

## deploy containerized tomcat to kube

### containerize the tomcat

1. ssh to the vm and run `docker ps` to find the tomcat container id

2. run `docker commit <container id> <new name>`

### login and push the image to acr

1. create acr in azure and go to access keys to enable admin user

2. ensure azure-cli is installed in the vm

3. run `sudo az login -u <username>`

4. run `sudo az acr --name <registry name>`, and enter the username and password when prompted

5. run `sudo docker login <login server>`, and enter the username and password if prompted

6. run `docker tag <image> <registry name>/<image>:<tag>` (if no tag is entered, latest tag will be assigned)

7. run `docker push <registry name>/<image>:<tag>` and check acr once completed

https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli?tabs=azure-cli

### pull and deploy from acr to kube

1. install minikube on local machine

2. run `docker login -u <username> -p <password> <registry>`

3. run `eval $(minikube -p minikube docker-env)`

4. pull image by running `docker pull 4cd467f3.azurecr.io/tomcat-deploy`, the image pulled should be useable on minikube

5. run `kubectl create namespace tomcat` to create the tomcat namespace

6. run `kubectl apply -f tomcat.yml` to start the pod

7. start the port forwarding using `kubectl port-forward main -n tomcat 8888:8080`

8. the port forwarding process will start and the terminal will not be useable during the port forwarding, type ctrl + c to cancel the port forwarding process

9. the tomcat app can be accessed from http://127.0.0.1:8888/simple-maven-war or http://127.0.0.1:8888/simple-maven-war/

### add secret using (if pull from acr)

```
kubectl create secret docker-registry <secret-name> --namespace <namespace> --docker-server=<container-registry-name>.azurecr.io --docker-username=<service-principal-ID> --docker-password=<service-principal-password>
```

example

```
kubectl create secret docker-registry acr-secret --docker-server=4cd467f3.azurecr.io --docker-username=4cd467f3 --docker-password=0D/RoYArTMYHgjDBLiln+oPOacenZUJc
```

https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-kubernetes

https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

### kubectl useful commands

```
kubectl create namespace <namespace name>

kubectl get secret

kubectl get namespace

kubectl get pods -n <namespace>

kubectl describe pod <pod name> -n <namespace>

kubectl apply -f <file>

kubectl delete -f <file>

kubectl get pod <pod> -n <namespace> -o wide

kubectl port-forward pod/<pod> -n <namespace> 8888:8888
```

### issues encountered

1. trouble creating principal account

no privileges

instead, docker login locally using `docker login -u <username> -p <password> <registry>` then pull the image

for example `docker login -u 4cd467f3 -p aZoQA+fcbxooMn6jeQf5rAk2wCgbX3Ii 4cd467f3.azurecr.io`

the username and password is from acr > access keys

2. pulling from registry

kubectl tried to pull the image from the registry instead of the local image

fixed it by adding `imagePullPolicy: Never` in the yaml file

3. ErrImageNeverPull 

need to run `eval $(minikube -p minikube docker-env)`

running `minikube docker-env` will outputs environment variables needed to point the local docker daemon to the minikube internal docker registry, and will propose to use `eval $(minikube -p minikube docker-env)`

then pull the image again

https://medium.com/swlh/how-to-run-locally-built-docker-images-in-kubernetes-b28fbc32cc1d

4. port forward connection refused

ran `kubectl port-forward main -n tomcat 8888:8888` and always connection refused

it is because i thought the container exposed port 8888, but actually port 8080 is the one being exposed. i thought this way because the tomcat image was using port 8888 during the deployment on azure devops before containerize. i thought it would change but it does not, because the port is only being forwarded from 8888 to 8080 during `docker run`. once it is containerized, the port forwarding stop so i need to expose 8080 instead of 8888 in kubectl.

once the command is changed to `kubectl port-forward main -n tomcat 8888:8080`, the pod can be accessed without any issue