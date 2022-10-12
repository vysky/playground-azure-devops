# playground-azure-devops

## quick start

1. start with spinning up a vm with docker, pull sonarqube and start it

2. create azure devops organization

3. git push the prepared repo to azure repos

4. install sonarqube extension through marketplace

5. configure and intergrate sonarqube and azure devops

6. create the pipeline and paste the yml code

---

## install sonarqube extension

1. go to azure devops, on the top right hand corner, find the marketplace icon

2. search for sonarqube and install it

---

## install sonarqube using vm through docker

1. create a vm and select ubuntu 20.04 image

2. ssh into the vm using `ssh -i <private key> <username>@<ip address>` and install docker using the commands below

```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

3. after docker is installed, run `docker pull sonarqube:lts`

4. once the image is pulled, run `docker run -d --name "sonarqube" -p 9000:9000 sonarqube:lts`

5. if encounter any permission issue, run with `sudo`

6. go to the vm > networking > inbound port rules, create a new rule and change the destination port ranges to 9000 (leave other values as default)

7. once the rule is added, sonarqube shoud be accessible using the vmip:9000

---

## integrate sonarqube and azure devops

1. go to azure devops and create a personal access token (save it for later use) with full access (should not for production use)

2. go to sonarqube > adminstration > alm integration > azure devops, paste the personal access token and url

3. go to projects > add project > azure devops, paste the personal token when prompted

4. select the project created > set up selected repository

5. select with azure pipelines when prompted and instructions on how to continue will appear

6. generate the token when prompted (save it for later use)

7. go back to azure devops > project settings (bottom left corner) > service connection > sonarqube > next

8. follow the instructions in sonarqube and enter the url, token and name

9. continue following instructions to edit the pipeline

---

## install sonarqube using container instances (but always crash since acloudguru only allow 1.5gb ram)

1. go to azure, search for container instances

2. in create container instances, select 'other registry' for image source and enter 'sonarqube:lts' unser image

3. go to networking, enter '9000' under port and select 'tcp' in the drop down list

4. create the instance and wait for it to complete

5. once created, find the ip address and access sonarqube using the ip address and the port, example '20.98.187.119:9000'

6. default username and password is `admin` for both

---

## create agent (put aside for now)

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

## sonarqube intergration

https://www.youtube.com/watch?v=GfgUTj4rp7c

---

## issues encountered

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