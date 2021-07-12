# feedify-Microservices

Cloud Microservices practice project on AWS

## Prerequisites
- kubeone v0.9.0 or newer, install it from https://github.com/kubermatic/kubeone
- Docker, install it from https://docs.docker.com/docker-for-windows/install/
- terraform v0.12.0 or later. install it from https://www.terraform.io/downloads.html

## Notes
1. my dockerhub repositories : https://hub.docker.com/u/abosamraa
2. after project deployment, Of course I deleted my IAM user in my aws account and all key pairs related to it to prevent access to my account , so all aws credentials or environment variables exposed in files will not working and you must replace it with your credentials.
## Steps

### 1. Creating docker images and push it to docker hub

1. After Docker installation , you need to sign up in docker hub from this link : https://hub.docker.com/

2. To login to your Docker hub account from your terminal, run :

    `sudo docker login --username=DockerHubUserName`

replace DockerHubUserName with your username

3. go to udacity-c3-restapi-feed directory from your terminal which include Dockerfile and run :  

    `sudo build -t udacity-restapi-feed . `

this will build udacity-restapi-feed image and save it locally.

3. Repeate previous step in udacity-c3-frontend and udacity-c3-restapi-user directory to create udacity-frontend and udacity-restapi-user images. repeate also in udacity-c3-deployment/docker for reverseproxy Dockerfile

4. then you should tag each image with this pattern:

    `sudo docker tag udacity-restapi-feed DockerHubUserName/udacity-restapi-feed`

replace DockerHubUserName with your username and repeate for each image.

5. Install Kubernetes using configuration output from Terraform:

    `kubeone install config.yaml --tfjson tf.json`

6. To push image to your Docker hub 
	- insure you are logged in  `docker login`
	- run. replace DockerHubUserName with your username
  
	    `sudo docker push DockerHubUserName/udacity-restapi-feed`


### 2. Docker compose

1. After pushing all images , go to udacity-c3-deployment/docker directory 

2. update all yaml files to point to your docker images instead of mine. you can search and replace 'abosamraa' (my docker hub username) to your username

3. change all your inveronment variables in .env file wich used by docker compose then run

    `sudo docker-compose up`


### 3. Creating Infrastructure with terraform 

1. In order for Terraform to successfully create the infrastructure and for KubeOne to install Kubernetes and create worker nodes you need an [IAM account](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) with the appropriate permissions.

    Once you have the IAM account you need to set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables:

    ```bash
    export AWS_ACCESS_KEY_ID=...
    export AWS_SECRET_ACCESS_KEY=...
    ```
2. Go to /udacity-c3-deployment/aws folder from your terminal
3. run command `terraform plan`
3. then run  `terraform apply`
Say `yes` to confirm creating the infrastructure
4. Create Terraform output file (will be used by KubeOne) 
`terraform output -json > tf.json`
5. Install Kubernetes using configuration output from Terraform: 
`kubeone install config.yaml --tfjson tf.json`
6. KUEBCONFIG file will be created. this help kubectl to know your configration you must run this command in any terminal window that the kubectl command will be run in. replace with your cluster name. make sure KUEBCONFIG file is in your current directory.
	- `export KUBECONFIG=$PWD/yourclustername-kubeconfig`

### 4. Setup Kubernetes environment
Open a new terminal within the project directory and follow instructions :

1. install kubectl command , follow this link : https://kubernetes.io/docs/tasks/tools/install-kubectl/
2. Use https://www.base64encode.org/ to generate encrypted values for aws credentials, Database User Name, and Database Password and put the values into aws-secret.yaml and env-secret.yaml files 
2. Load secret files: 
	- `kubectl apply -f aws-secret.yaml`
	- `kubectl apply -f env-secret.yaml`
3. Load config map: 
    - `kubectl apply -f env-configmap.yaml`
4. Apply Deployments:
	- `kubectl apply -f backend-feed-deployment.yaml`
	- `kubectl apply -f frontend-deployment.yaml`
	- `kubectl apply -f backend-user-deployment.yaml`
5. Apply Services:
	- `kubectl apply -f backend-feed-service.yaml`
	- `kubectl apply -f backend-user-service.yaml`
	- `kubectl apply -f frontend-service.yaml`
6. Deploy reverse proxy, has to be done after the services are running:
	- `kubectl apply -f reverseproxy-deployment.yaml`
	- `kubectl apply -f reverseproxy-service.yaml`
7. Perform port forwarding (each needs to be run in a separate terminal window and left running)
	- `kubectl port-forward service/frontend 8100:8100`
	- `kubectl port-forward service/reverseproxy 8080:8080`
9. go to http://localhost:8100 at your browser to check application is working
10. run these commands to check cluster status : 
    - `kubectl get nodes`
    - `kubectl get pod --all-namespaces`
    - `kubectl get svc`
    - `kubectl get configmaps`
    - `kubectl get secrets`
    - `kubectl describe secret/env-secret`

### 5. Continuous Integration / Continuous Development:
1. We will use Travis CI to automatically build and deploy the Docker containers.
2. Sign up in Travis CI and follow this tutorial https://docs.travis-ci.com/user/tutorial/
3. go to setting in Travis CI and add DOCKER_PASSWORD and DOCKER_USERNAME environment variables
4. to build docker images, follow this tutorial https://docs.travis-ci.com/user/docker/

### 6. Rolling updates  :
1. follow these instructions in kubernetes :
https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/

### 7. AWS Cloudwatch kubernetes Log :
1. follow these instructions in aws
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html .

Congratulations! You're all set .
### (Optional) Delete cluster and Infrastructure at aws :

1. follow this tutorial from aws :
https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html
2. you can destroy Infrastructure by running :
`terraform destroy`
this command may not be able to delete every thing in aws , so you should check your aws account and delete any running resources manually to avoid extra charges .
