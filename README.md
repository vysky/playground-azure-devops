# playground-azure-devops

## summary

1. vm

	a. create vm and install docker
	
	b. pull and run sonarqube

	c. build tomcat with dockerfile and run the containerized image

	d. add inbound port rules

2. azure devops
	
	a. create azure devops organization

	b. push a git repo to azure repos in azure devops

	c. install tomcat and sonarqube extensions

	d. setup and integrate sonarqube with azure devops

	e. run azure devops pipeline
	
		i. scan source code using sonarqube

		ii. build the artifact using maven
		
		iii. deploy the .war file to tomcat

3. kubernetes

	a. install minikube on local machine

	b. commit (update) the tomcat container with the deployed .war file

	c. push the updated tomcat image to acr
	
	d. pull the updated tomcat image from acr to local machine
	
	e. create pod using kubectl (kubernetes command line tool) through minikube
	
	f. run port forwarding on the pod so tomcat will be accessible locally

## vm

### create a vm

1. go to azure portal and create a vm using ubuntu 20.04 server image

2. ssh into the vm using `ssh -i <private key> <username>@<ip address>`

### install docker

1. install docker using the following commands

```
sudo apt-get update

sudo apt-get install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### pull and run sonarqube

1. run `sudo docker pull sonarqube:lts`

2. once the image is pulled, run `sudo docker run -d --name "sonarqube" -p 9000:9000 sonarqube:lts` to start the sonarqube container

3. run `sudo docker ps` to list all running containers

4. tip - run `sudo docker ps --all` to list all containers

### build and run tomcat

1. git clone this repo in the vm

2. cd into the dockerfile directory of the repo

3. run `sudo docker build tomcat -t "tomcat"` to build the tomcat image

4. run `sudo docker images` to list all top level images

5. run `sudo docker run -d --name "tomcat" -p 80:8080 -p 8888:8080 tomcat` to start the tomcat container

6. tip - adding `-p 80:8080` in `docker run` enables the container to be accessible on http (without adding the port behind the ip address), and `-p 443:8080` for https

	https://stackoverflow.com/a/73841213

### add inbound port rules

1. go back to azure portal > vm created in previous step > networking > inbound port rules

2. create a new rule and change the destination port ranges to 80 (leave other values as default)

3. repeat for port 8888 and 9000 (or the ports you put for docker run the previous steps)

## azure devops

### create azure devops organization

1. create azure devops organization if new (or if you are using acloudguru's cloud playground like i am)

2. create a project in azure devops

### adding repo to azure devops

1. on your local machine, git clone this repo

2. cd into the repo, run `rm -rf .git` to remove all the git information

3. run `git init`, `git config user.email <email>`, `git config user.name <username>` to setup the git repo

4. run `git remote add origin <azure repo>`, example of azure repo https://clouduserp9583c7fe@dev.azure.com/clouduserp9583c7fe/cicd/_git/test

	the commands can be found when creating a new project and trying to setup the repo

5. run `git commit -m "<message>"`

6. run `git push -u origin --all` to push the code to azure repo

7. going back to azure devops > repos, refreshing the page should show the pushed code

### install apache tomcat deployment extension

1. go to azure devops and click on the marketplace icon (top right hand corner)

2. search for apache tomcat deployment and install the extension

	https://marketplace.visualstudio.com/items?itemName=ms-vscs-rm.apachetomcat

### install sonarqube extension

1. go to azure devops and click on the marketplace icon (top right hand corner)

2. search for sonarqube and install the extension

	https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube

### integrate sonarqube with azure devops

#### personal access token

1. go to azure devops, click on the user settings icon (beside the user profile pic) > personal access token > new token

2. create a token with full access and save the token for later use

3. go back to sonarqube > adminstration > alm integration > azure devops, paste the personal access token and the azure devops url (example https://dev.azure.com/clouduserp9583c7fe) when prompted

#### service connection

1. in sonarqube, go to projects > add project > azure devops

2. paste the personal token when prompted

3. select the project created previously in azure devops > set up selected repository

4. select with azure pipelines when prompted and instructions on how to continue will appear

5. generate the token (step 2) when prompted and save it for later use (you can also generate the token from adminstrative > security > users > token icon)

6. go back to azure devops > project created previously > project settings (bottom left corner) > service connection > create service connection > sonarqube > next

7. follow the instructions in sonarqube and enter the url, token and name > save the service connection

### run the pipeline

1. before running the pipeline, ensure the azure-pipelines.yml file is udated with the vm ip address

2. to run the pipeline, go back to azure devops > pipelines > select azure repos git > the project created previously

3. azure devops should auto detect the azure-pipelines.yml file

4. click run pipeline to run the pipeline

5. once the pipeline is finished, check tomcat url (<url>:<port>/simple-maven-war) to find the app accessible with the deployed .war file

## kubernetes

### commit (update) tomcat container with the deployed .war file

1. go back to the vm and run `sudo docker ps` to find the tomcat container id

2. run `sudo docker commit <container id> <new name>` to update the tomcat image

3. run `sudo docker images` to find the update tomcat iamge

### login and push the image to acr

1. go back to azure portal, create a azure container registry and go to access keys page to enable admin user

2. go back to the vm and install azure-cli using `sudo apt install azure-cli`

3. once azure-cli is installed, run `sudo az login -u <username>` and enter the password when prompted (azure user account, not the admin account created in step 1)

4. run `sudo az acr login --name <registry name>` to login to acr, enter the username and password when prompted (the admin user username and password generated from step 1)

5. run `sudo docker tag <image> <login server>/<image>:<tag>`

6. run `sudo docker push <registry name>/<image>:<tag>` and go back to azure portal to check acr once completed

	https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli?tabs=azure-cli

### pull the tomcat image from acr to local machine

1. install minikube on local machine

2. run `docker login -u <username> -p <password> <login server>` to login to acr

3. run `eval $(minikube -p minikube docker-env)` to point the local docker daemon to the minikube internal docker registry

4. pull image by running `docker pull <login server>/<image name>:<tag>`, the image pulled will be useable on minikube

### kubectl apply and port forwarding

1. run `kubectl create namespace tomcat` to create the tomcat namespace

2. run `kubectl apply -f tomcat.yml` to start the pod

3. start the port forwarding using `kubectl port-forward main -n tomcat 8888:8080`

4. the port forwarding process will start and the terminal will not be useable during the port forwarding, type ctrl + c to cancel the port forwarding process

5. the tomcat app can be accessed from http://127.0.0.1:8888/simple-maven-war or http://localhost:8888/simple-maven-war/

### create secret for acrs (if login and pull from acr instead of docker)

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

## sonarqube

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

### resources

https://www.youtube.com/watch?v=GfgUTj4rp7c

https://www.youtube.com/watch?v=ApMeWoRkv-A

https://docs.sonarqube.org/latest/analysis/azuredevops-integration/

## tomcat

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

## minikube / kubernetes

### issues encountered

1. trouble creating principal account

wanted to pull the image from acr but no privileges

instead, docker login locally using `docker login -u <username> -p <password> <registry>` then pull the image

for example `docker login -u 4cd467f3 -p aZoQA+fcbxooMn6jeQf5rAk2wCgbX3Ii 4cd467f3.azurecr.io`

the username and password is from acr > access keys

2. pulling from registry

kubectl tried to pull the image from the registry (fail as the account provided do not have enough privilges) instead of using the local image

fixed it by adding `imagePullPolicy: Never` in the yaml file

3. ErrImageNeverPull 

need to run `eval $(minikube -p minikube docker-env)`

running `minikube docker-env` will outputs environment variables needed to point the local docker daemon to the minikube internal docker registry, and minikube will propose to use `eval $(minikube -p minikube docker-env)`

run the command then pull the image again, the image will be pulled to minikube internal docker registry instad of the docker on the local machine (wsl integration)

https://medium.com/swlh/how-to-run-locally-built-docker-images-in-kubernetes-b28fbc32cc1d

4. port forward connection refused

ran `kubectl port-forward main -n tomcat 8888:8888` and always connection refused

it is because i thought the container exposed port 8888, but actually port 8080 is the one being exposed. i thought this way because the tomcat image was using port 8888 during the deployment on azure devops before containerize. i thought it would change but it does not, because the port is only being forwarded from 8888 to 8080 during `docker run`. once it is containerized, the port forwarding stop so i need to expose 8080 instead of 8888 in kubectl.

once the command is changed to `kubectl port-forward main -n tomcat 8888:8080`, the pod can be accessed without any issue