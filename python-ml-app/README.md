# GitLab Repo -> https://gitlab.com/kapil.kumaria/insurance-fraud-detection
# Insurance Fraud Detection
--------------------------------------------------------------------------------------------------------------
    1.1    For Testing - Using Docker Compose:
    1.2    For Production - Using Jenkins, Kubernetes, and ArgoCD

--------------------------------------------------------------------------------------------------------------
# PROJECT SNAPSHOT

## For Testing - Using Docker Compose:

    1.1    Spin up EC2 instance – Correct.
    1.2    Install Docker – Correct.
    1.3    Install Docker Compose – Correct, since Docker Compose is perfect for local testing without needing a full Kubernetes cluster.
    1.4    Scan the Python code using Sonar – Correct. Ensure the SonarQube server is set up locally or available for scanning. 
           Once the code passes the quality gate, move to the next step.
    1.5    Build Docker image using Dockerfile – Correct. Build the Docker image for your Insurance Fraud Detection Python project.
    1.6    Scan this image using Trivy – Correct. Trivy will ensure the image is secure before pushing it to AWS ECR or JFrog Artifactory
    1.7    Push the image to AWS ECR – Correct, push the image to AWS ECR once the scan passes.
           or
           Push the image to JFrog Artifactory – Correct, push the image to JFRog Artifactory once the scan passes.
    1.8    Use the ECR image in Kubernetes manifests – Correct. In your Kubernetes manifests, reference the image stored in AWS ECR for deployment.
           or
           Use the JFrog Artifactory in Kubernetes manifests – Correct. In your Kubernetes manifests, reference the image stored in JFrog Artifactory for deployment.
    1.9    Access the app in the browser – Correct. You’ll expose your service and be able to access the app via browser once it’s running.
    1.10   Check database entries – Correct. You can access the database within the Kubernetes pods to verify data after interacting with the app.

--------------------------------------------------------------------------------------------------------------
# 1st EC2 Machine
--------------------------------------------------------------------------------------------------------------
## ML-EC2 machine Ubuntu 22.04 - t3.large
--------------------------------------------------------------------------------------------------------------
On my 1st ec2 ubuntu, I will clone a GitLab repo. This is machine learning model with python code. 
and I will create 2 docker images and 2 docker conatiners using Dockerfile and images will be 'web' and 'db' 

## Using docker-compose file.
## Install docker and docker-compose
```
sudo apt install git
sudo apt install docker.io -y
sudo apt-get install docker.io docker-compose-v2
docker ps
sudo usermod -aG docker $USER
docker ps
docker compose
sudo reboot
docker ps
docker compose
```
--------------------------------------------------------------------------------------------------------------
## Check out the GitLab Repo + Deploy Application and Database
```
git clone https://github.com/jargans/insurance-fraud-detection.git
cd insurance-fraud-detection/
docker compose up -d
docker ps
```
- see 2 docker conatiners (insurance-fraud-detection-web-1 & insurance-fraud-detection-db-1)
- Access the application via browser -> <Pubic IP of EC2> e.g <http://44.222.139.120:5000/>
- Register or Login and then verify users in db container below . . .
```
docker exec -it insurance-fraud-detection-db-1 bash
> mysql -u root -p    ------------------------------------------------??????
> show databases;
> use .....;
> select * from users;
> exit
```
SUCCESS

# Testing with Deploying application + MYSQL database in Minikube Cluster

## Setup Kubernetes Cluster for Testing:
## Minikube Setup:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start
```
# Install Kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl version --client --output=yaml
   
alias k=kubectl
k get nodes
k get pods
k get ns
k get all
k get all -A
```
   
> docker-compose command created docker images for 'web' and 'db'

# By the way here is our Dockerfile:

```
# Use a lightweight Python image
FROM python:3.9-slim

# Install system dependencies
RUN apt-get update \
    && apt-get install -y \
        gcc \
        libmariadb-dev \
        pkg-config \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements file and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the Flask app code and initialization script
COPY . .

# Expose port 5000
EXPOSE 5000

# Command to run the Flask application
CMD ["sh", "-c", "python init_db.py && python app.py"]
```

# By the way here is our docker-compose.yaml:
```
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: flaskapp
      MYSQL_USER: flaskuser
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      retries: 5
      start_period: 30s

  web:
    build:
      context: .
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: flaskapp
      MYSQL_USER: flaskuser
      MYSQL_PASSWORD: password
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```
--------------------------------------
```
docker compose up -d
```
> This above command will create 2 images and 2 containers

# Tagging Docker Images to Push them To DockerHub

> We will tag those images so that we can push these images to DockerHub:
```
docker tag insurance-fraud-detection-web kappu1512/insurance-fraud-detection-web
docker images
docker tag mysql:8.0 kappu1512/mysql:8.0
docker login
docker push kappu1512/insurance-fraud-detection-web
docker push kappu1512/mysql:8.0
```  
> Since we have these images in DockerHub, we can pull these images in K8s manifests files:
  
- Login to DockerHub and verify these images

# Tagging Docker Images To Push them to AWS ECR:
1. Create ECR 2 repositories in console
   1.1 insurance-fraud-detection-app
   1.2 insurance-fraud-detection-db
2. Check docker images
   ```
   docker images
   ```
3. Tag docker images so that it can be pushed to ECR
   ```
   docker tag insurance-fraud-detection-web <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-app:v1.0
   docker tag mysql:8.0 <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-db:v1.0
   ```
4. Verify the images has been tagged
   ```
   docker images
   ```
5. Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
   ```
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-number>.dkr.ecr.us-east-1.amazonaws.com (get this command from ECR - Push Commands)
   Login suceeded
   ```
6. Authenticate Kubernetes to AWS ECR: Use AWS CLI to generate an authentication token that Kubernetes will use to pull the Docker images. This is done by creating a Kubernetes secret using the following command:

   ```
   kubectl create secret docker-registry aws-ecr-secret --docker-server=<account-number>.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-east-1) --docker-email=kapil.kumaria@gmail.com
   ```
7. Update Your Kubernetes Deployment YAML File: Update your deployment YAML file to reference the newly created secret for pulling images. You can do this by specifying the imagePullSecrets field in your YAML:
    Here are the updated K8s deployment manifests for web and mysql db
    
## deployment-web.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: flask-app
#          image: kappu1512/insurance-fraud-detection-web:latest
          image: <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-app:v1.0
          ports:
            - containerPort: 5000
          env:
            - name: MYSQL_HOST
              value: "db-service"
            - name: MYSQL_DATABASE
              value: "flaskapp"
            - name: MYSQL_USER
              value: "flaskuser"
            - name: MYSQL_PASSWORD
              value: "password"
      imagePullSecrets:
      - name: aws-ecr-secret
    
```

## deployment-mysql.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: mysql
#          image: kappu1512/mysql:8.0
          image: <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-db:v1.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "password"
            - name: MYSQL_DATABASE
              value: "flaskapp"
            - name: MYSQL_USER
              value: "flaskuser"
            - name: MYSQL_PASSWORD
              value: "password"
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
          livenessProbe:
            exec:
              command: ["mysqladmin", "ping", "-h", "localhost", "-u", "root", "-ppassword"]
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command: ["mysqladmin", "ping", "-h", "localhost", "-u", "root", "-ppassword"]
            initialDelaySeconds: 30
            periodSeconds: 10
      imagePullSecrets:
      - name: aws-ecr-secret
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: db-pvc
```
         
# Create K8s manifests using DockerHub Docker Images:
(We have alreday created above, but see once more . . .)
---------------------------
vi deployment-mysql.yaml 
---------------------------
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: mysql
          image: kappu1512/mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "password"
            - name: MYSQL_DATABASE
              value: "flaskapp"
            - name: MYSQL_USER
              value: "flaskuser"
            - name: MYSQL_PASSWORD
              value: "password"
      imagePullSecrets:
      - name: aws-ecr-secret
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
          livenessProbe:
            exec:
              command: ["mysqladmin", "ping", "-h", "localhost", "-u", "root", "-ppassword"]
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command: ["mysqladmin", "ping", "-h", "localhost", "-u", "root", "-ppassword"]
            initialDelaySeconds: 30
            periodSeconds: 10
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: db-pvc
```            
---------------------------
vi deployment-web.yaml 
---------------------------
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: flask-app
          image: kappu1512/insurance-fraud-detection-web:latest
          ports:
            - containerPort: 5000
          env:
            - name: MYSQL_HOST
              value: "db-service"
            - name: MYSQL_DATABASE
              value: "flaskapp"
            - name: MYSQL_USER
              value: "flaskuser"
            - name: MYSQL_PASSWORD
              value: "password"
      imagePullSecrets:
      - name: aws-ecr-secret
```              
---------------------------
vi pvc.yaml 
---------------------------
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```      
---------------------------
vi service-mysql.yaml 
---------------------------
```
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: db
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```    
---------------------------
vi service-web.yaml 
---------------------------      
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: LoadBalancer
```
-----------------------------------------
> So we created these 5 K8s manifests:

    1.1 deployment-mysql.yaml
    1.2 deployment-web.yaml
    1.3 pvc.yaml
    1.4 service-mysql.yaml
    1.5 service-web.yaml
-----------------------------------------

- cd to k8s folder and apply all the manifest files:
```
kubectl apply -f .
k get all
k get pods
```
SUCCESS

## Browse the application in the browser, 

  - we cannot use LoadBalancer because running Minikube and don’t have a cloud load balancer

## Port Forwarding

1. Accessing via SSH Tunnel

   You can forward ports from the EC2 instance to your local machine using an SSH tunnel. This way, you will still use localhost, but the traffic will be routed through SSH to your EC2 instance.

## Here's how to set it up:

   SSH into EC2 with Port Forwarding:

   Assuming you have SSH access to your EC2 instance, run this command from your local machine:

```
ssh -L 5000:localhost:5000 -i your-ec2-key.pem ubuntu@<your-ec2-public-ip>

# Replace <your-ec2-public-ip> with the public IP of your EC2 instance and your-ec2-key.pem with your private key.
```

# Run the kubectl port-forward Command:

Now, on your EC2 instance (via the SSH session), run the kubectl port-forward command:

```
kubectl port-forward service/web-service 5000:5000
```
# Access in the Browser:

On your local machine, open your browser and navigate to:

```
http://localhost:5000
```

The SSH tunnel will forward traffic from your local machine to the Kubernetes service running inside the EC2 instance.

SUCCESS

--------------------------------------------------------------------------------------------------------------
# Sonar Scanner Installtion and Static Code Scanning
## Download SonarScanner

```
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-linux.zip
```
## Unzip the file
```
sudo apt install unzip -y
unzip sonar-scanner-cli-4.6.2.2472-linux.zip
```
# Move to a suitable location (e.g., /opt)
```
sudo mv sonar-scanner-4.6.2.2472-linux /opt/sonar-scanner
```
# Add SonarScanner to the PATH
```
export PATH=$PATH:/opt/sonar-scanner/bin
cd insurance-fraud-detection/
```
```
nano sonar-project.properties
```
> The sonar-project.properties file is essential for configuring the details of your project when running a SonarQube scan.
> In a Python project (or any other project), this file defines the parameters SonarQube needs to perform static code analysis on your project
> and report back code quality, security vulnerabilities, and code smells.

```
  sonar.projectKey=your-project-key
  sonar.projectName=insurance-fraud-detection
  sonar.projectVersion=1.0
  sonar.sources=.
  sonar.language=py
  sonar.sourceEncoding=UTF-8
  sonar.host.url=<SonarQube URL> e.g. <http://3.237.103.124:9000>
  sonar.login=squ_91aead193f9cfec511254a83b07cd359e856895d
  sonar.python.coverage.reportPaths=coverage.xml
  # If you're using a requirements.txt or any other dependency file
  sonar.python.dependencies=requirements.txt
  # Optional if you want to exclude some directories (like virtual environments)
  sonar.exclusions=**/venv/**
```
```  
sonar-scanner
# See the result in SonarQube Dashboard
```

SUCCESS

--------------------------------------------------------------------------------------------------------------
# Install Trivy and Vulnerabilty and Security Testing for Docker Images (App + db)
```
sudo apt-get update
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
trivy
```
# Scan Docker Images for Vulnerabilities and Security
```
docker images
trivy image mysql:8.0
trivy image insurance-fraud-detection-web
```
SUCCESS

--------------------------------------------------------------------------------------------------------------
# Install JFrog CLI, Configure, Login to JFrog and Pushing Docker Images to JFrog Artifactory
```
curl -fL https://getcli.jfrog.io | sh
sudo mv jfrog /usr/local/bin/
jfrog config add artifactory-server-kk --artifactory-url=http://3.231.33.186:8082/artifactory
jfrog rt config
jfrog rt ping
jfrog rt docker-login
```
>> PROBLEMS, SO ADDING Artifactory server's IP address to the list of insecure-registries, below . . . 

## Add your Artifactory server's IP address to the list of insecure-registries:
```
sudo nano /etc/docker/daemon.json
add -> 
{
  "insecure-registries": ["3.238.31.216:8082"] # This is URL of JFrog Artifactory
}
```
```
sudo systemctl restart docker
```
## Login to JFrog Artifactory
```
docker login 3.238.31.216:8082 -u admin -p @Kapil1512
```
## Docker Images
```
docker images
docker ps
```
## Docker tag for JFrog Artifactory  
```
docker tag mysql:8.0 3.238.31.216:8082/docker-local/mysql:8.0
docker tag insurance-fraud-detection-web:latest 3.238.31.216:8082/docker-local/insurance-fraud-detection-web:latest 
```
> Here,
    '3.238.31.216:8082' is the URL of Artifactory
    'docker-local' is the name of repository in JFrog
    'insurance-fraud-detection-web:latest' is the original docker image name
    '3.238.31.216:8082/docker-local/insurance-fraud-detection-web:latest' is the name of image to push it to JFrog Artifactory
## Push Docker Images to JFrog Artifactory
```
docker push 3.238.31.216:8082/docker-local/insurance-fraud-detection-web
```
--------------------------------------------------------------------------------------------------------------
# Creating AWS ECR Repositories and Pushing Docker Images to these Repositories

# Creating IAM Role, so that using this Role, Docker can push images to AWS ECR

- Create a IAM Role: EC2-ECR-Push-Role and attach AWS Managed Policy: AmazonEC2ContainerRegistryFullAccess
- Attach this Role to this EC2 Instance

# ECR
Create 2 Repositories in ECR,
   1.1 insurance-fraud-detection-app [for storing python app docker images]
   1.2 insurance-fraud-detection-db [for storing python db docker images]

# Docker images
      we have 2 images - insurance-fraud-detection-web:latest & mysql:8.0 (see the names in docker-compose.yml file in the GitLab repo)
      View Push Commands in the ECR 2 Repositories

## Pushing the 'app' docker image to ECR:
Use the following steps to authenticate and push an image to your repository. For additional registry authentication methods, including the Amazon ECR credential helper, see Registry Authentication 

## Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-number>.dkr.ecr.us-east-1.amazonaws.com
Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.
```
## Tag your image so you can push the image to this repository:
```
docker tag insurance-fraud-detection-app:latest <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-app:latest
```
## Run the following command to push this image to your newly created AWS repository:
```
docker push <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-app:latest
```
## Pushing the 'db' docker image to ECR:
Use the following steps to authenticate and push an image to your repository. For additional registry authentication methods, including the Amazon ECR credential helper, see Registry Authentication 

Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-number>.dkr.ecr.us-east-1.amazonaws.com
Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.
```
## Tag your image so you can push the image to this repository:
```
docker tag insurance-fraud-detection-db:latest <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-db:latest
```
# Run the following command to push this image to your newly created AWS repository:
```
docker push <account-number>.dkr.ecr.us-east-1.amazonaws.com/insurance-fraud-detection-db:latest
```
SUCCESS

--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
2nd EC2 Machine
--------------------------------------------------------------------------------------------------------------
JFrog-ML -> EC2 Ubuntu 22.04 t2.medium
--------------------------------------------------------------------------------------------------------------
# Jfrog Artifactory SetUp

## Steps to Set Up JFrog Artifactory on an EC2 Ubuntu Instance

Step 1: SSH into the EC2 Instance

Step 2: Use the following command to SSH into your Ubuntu EC2 instance,
```
ssh -i "your-key.pem" ubuntu@your-instance-public-ip
```
## Allow inbound traffic port 22, 8081 and 8082 from your IP 

## Important Note on EC2 Configuration

Note: When setting up Jenkins on an EC2 instance, it is highly recommended to associate an Elastic IP with the instance. By default, the public IP of your EC2 
      instance changes every time the instance is stopped and started. If you don't use an Elastic IP, you'll need to update the Jenkins URL in 
      Jenkins > Manage Jenkins > Configure System > Jenkins URL every time, which can cause Jenkins to behave slowly or cause connectivity issues.
To avoid this, allocate an Elastic IP in AWS and associate it with your EC2 instance, ensuring that your Jenkins instance always remains accessible at the same IP address.

Step 3: Install Java
```
# JFrog Artifactory requires Java to run. Use the following commands to install OpenJDK 17,

sudo apt update
sudo apt install openjdk-17-jre -y
java -version
```
Step 4: Download and Extract JFrog Artifactory

```
Download the latest version of JFrog Artifactory from the official release repository,

sudo su
wget https://releases.jfrog.io/artifactory/artifactory-pro/org/artifactory/pro/jfrog-artifactory-pro/7.55.14/jfrog-artifactory-pro-7.55.14-linux.tar.gz
tar -xvzf jfrog-artifactory-pro-7.55.14-linux.tar.gz
```
Step 5: Navigate into the Extracted Directory

```
cd artifactory-pro-7.55.14/
cd app/bin/
```

Step 6: Install JFrog Artifactory as a Service
```
# To install JFrog Artifactory as a service, run the installation script,

 ./installService.sh
```
Step 7: Install net-tools package:
```
# Run the following command to install the missing net-tools package:
```
sudo apt update
sudo apt-get install net-tools
```
Step 8: Configure Firewall Rules
```
# Ensure the necessary ports are open for JFrog Artifactory,

```
ufw allow 8081
ufw allow 8082
```
Step 9: Start and Check Artifactory Service
```
# Start the Artifactory service using the systemctl command,
systemctl start artifactory.service
```
Step 10: Access JFrog Artifactory
```
# Start the Xray service:
  sudo systemctl start xray.service
```
Step 11: Access JFrog Artifactory
```
# Once the service is up and running, JFrog Artifactory will be accessible via the following ports,
8081: Artifactory
8082: API Access
```
## You can access the JFrog Artifactory UI by navigating to http://<server-ip>:8081.

# JFrog Artifactory UI Setup

> After successfully installing and starting JFrog Artifactory, follow these steps to configure your local repository using the JFrog UI.

Step 1: Access JFrog Artifactory
```
# Open your browser and navigate to the JFrog Artifactory UI,

http://<server-ip>:8081
```
```
# Log in using the default credentials or your configured admin account
```
Step 2: Create a Local Maven Repository

```
# Once logged in, follow these steps to create a local repository for Maven artifacts:

     1.1 In the top menu, click on Administration.
     1.2 In the left sidebar, select Repositories.
     1.3 Click on Add Repository and choose Local Repository from the options.
     1.4 Select Maven as the Package Type.
     1.5 In the Repository Key field, enter an identifier or name for your repository (e.g., maven-local).
     1.6 Review any other configuration options you may want to adjust.
     1.7 Click Create to set up your local Maven repository.
```

Step 3: View Artifacts and Binaries
```
# To view the artifacts and binaries uploaded to your repository:

1.1 Go to the Application section in the JFrog UI.
1.2 Navigate to Artifactory > Artifacts.
1.3 Here, you'll be able to see all the artifacts and binaries stored in your newly created local Maven repository.
```
SUCCESS

--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
# 3rd EC2 Machine
--------------------------------------------------------------------------------------------------------------
# SonarQube-ML -> EC2 Ubuntu 22.04 t2.medium
--------------------------------------------------------------------------------------------------------------
# SonarQube Installation & SetUp
--------------------------------------------------------------------------------------------------------------
```
sudo apt update
sudo apt install openjdk-17-jre -y
java -version
```
```
sudo su -
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.6.0.92116.zip
sudo apt install unzip -y
ll
unzip sonarqube-10.6.0.92116.zip 
mv sonarqube-10.6.0.92116 sonarqube
ll
rm sonarqube-10.6.0.92116.zip 
ll
useradd sonaradmin
chown -R sonaradmin:sonaradmin /opt/sonarqube
ll
sudo su - sonaradmin
> cd /opt/sonarqube/bin/linux-x86-64
> ls
> ./sonar.sh status
> .sonar.sh start
```
# Open SonarQube Port in Security Group
```
# Ensure that port 9000 is open in the EC2 instance's security group to allow access to the SonarQube web interface.

sudo ufw allow 9000
```
# Access SonarQube in Browser
```
# Once the service is running, you can access SonarQube via a web browser at,

http://<Public-IP>:9000
```
# Generate Token

Administration > Security > User > Generate Token > Save this Token

SUCCESS

--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------



