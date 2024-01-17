## Containerizing Spring Boot application through Jenkins Pipeline

### Prerequisites :
1.  EC2 instance 
    OS : Amazon Linux 2
    Type : t2.micro

2. Tools used : Git, Maven, Docker, Docker Compose, Jenkins

3. Tech : MySQL and Java

The next steps for the task are:

#### 1. Install Jenkins: You can install Jenkins on EC2 instance by following these commands:

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins -y

#### 2. Start Jenkins: Once installed, you can start Jenkins with the following command:
sudo systemctl start jenkins

#### 3. Enable Jenkins: By default, Jenkins is not enabled to start at boot. You can enable Jenkins by running the following command:
sudo systemctl enable jenkins

#### 4. Give sudo powers to jenkins and put him in docker group
sudo vim /etc/sudoers

 
#### 5. Install docker using command
yum install docker -y

#### 6. Run the commands one after another to install compose
```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)"  -o /usr/local/bin/docker-compose
sudo mv /usr/local/bin/docker-compose /usr/bin/docker-compose

sudo chmod +x /usr/bin/docker-compose
```
     

#### 7. Configure Jenkins: After starting Jenkins, you can access the Jenkins web interface by navigating to <Public_IP_Address_of EC2_Instance:8080>. You will need to follow the on-screen instructions to configure Jenkins, including creating an admin user and installing plugins.

 

```
We need to add Inbound rule 8080 in our instances’s security group so that we can access Jenkins server.
```
#### Generating GitHub credentials

 1. Click on your profile icon and select "Settings" from the dropdown menu.

 2.   From the left sidebar, click on "Developer settings".

 3.   Click on "Personal access tokens".

 4.  Click on the "Generate new token" button.

 5. Select the scopes you need for your token. Scopes allow you to define the level of access you want to grant to the token. You can select the default scopes or customize them based on your needs.

 6.  Click on the "Generate token" button at the bottom of the page.

 7. Copy the generated token to a safe place, as it won't be displayed again.

#### Now go to Jenkins Server:

  1. Open the Jenkins web interface and navigate to the Manage Jenkins page.

  2.  Click on the Credentials link, then click on the System link.

  3.  Click on the Global credentials (unrestricted) link.

  4.  Click on the Add Credentials link to add new credentials.

  5.  Give your GitHub username and Personal token access (classic) not your password and save.


 
## Creating a pipeline

Now we have to create a Jenkins file, It describe the pipeline for a project. It also defines different stages of the pipeline, like build, test and, deploy.

#### Benefits of using Jenkins file:

*  Version control – You can track changes and roll back to previous version if necessary.

*    Code review – It can be reviewed by your team before executing. To avoid any mistakes and changes made.
```
pipeline {
    agent any
    tools {
        maven 'maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'ram', credentialsId: 'springboot-ram', url: 'https://github.com/cerebrone-ai/Cerebrone-training.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }


        stage('Deploy') {
            steps {
                sh 'sudo docker-compose up -d'
            }
        }
    
       stage('Check container status') {
           steps {
               sh 'sudo docker ps'
           }
       }
       stage('Test') {
            steps{
                    sh '''
                       #!/bin/bash
                       url='http://3.94.211.73:9191/'
                       code=`curl -sL --connect-timeout 20 --max-time 30 -w "%{http_code}\\n" "$url" -o /dev/null`
                       if [[ $code == 200 ]]; then
                       echo "Website $url is online."
                       else
                       exit 1
                       fi
                    '''
            }
        }    
        
    }   
}

    
```
Add this file and docker-compose file to the GitHub repo.

* This is the Docker-compose file we created, it creates two containers one for the application and the another one for database.
```
version: "3"
services:
  app:
    build:
    context: .
    dockerfile: dockerfile
    ports:
      - "9191:9191"
    networks:
      - spring
    depends_on:
      - db

  db:
    image: mysql:8.0
    networks:
      - spring
    environment:
      - MYSQL_ROOT_PASSWORD=ram2001
      - MYSQL_DATABASE=TestTable

networks:
  spring:
```       

* This is the docker file we used, the main benefit of using dockerfile in docker compose is it makes it easier to maintain and update your application. Any changes in the docker file will reflect across all the containers that use it.
```
FROM openjdk:11-jdk-slim

COPY target/spring-boot-crud-example-0.0.1-SNAPSHOT.jar /app/my-app.jar

WORKDIR /app

    CMD ["java", "-jar", "my-app.jar"]
```
  * Now we have to create a pipeline go to jenkins dashboard and select new item, give your pipeline a name and select pipeline and click on ok.


   *  Now in pipeline definition select pipeline script form SCM 


   * Now in SCM select Git and give you repo URL and credentials we created earlier and your branch name.


And click on save.

  * Go to your pipeline and click on build to run the pipeline on a successful build you will see this.

Screenshot (515).png

* You can check if the application is running or not by running this 
```
    localhost:9191/start
```
you will see a welcome page like this

 
### Blockers

 1.) GitHub connection 

  * Solution:  Check the git path in Manage credentials –> Global tool configuration –> Git

  *  Now in path to Git executable give git and click on save.

  * Also make sure you created the credentials using PAT instead of password and change the branch name form master to your branch name. 

 

2.) Database connection error


* Solution: Make sure you mentioned the correct database container name in the application.properties file instead of local host because our application is connecting to the database container not to the local mysql server.

 * And if the container is exiting after the creation check the container logs if database container connection  is lost and path to the jar file in docker-compose file. Give the right path to the jar file.

 

 

#### Exercise: Creating a Jenkins Pipeline for Docker Compose Deployment

Instructions:

1. Use the GitHub repository used in the previous exercise to your GitHub account.
    The same you pushed on github with your branch

2. Set up a Jenkins server or use an existing Jenkins instance. (http://jenkins.labcerebrone.com/)

3. Install necessary plugins in Jenkins. 

4. Make sure you install docker in other EC2 server and Integrate Docker with Jenkins server.

5. Create a new Jenkins pipeline job.

6. Configure the pipeline job to fetch the source code from your GitHub repository.

7. Create a Jenkinsfile in the root directory of the repository to define the pipeline stages.

8. Define the necessary stages in the Jenkinsfile to achieve the following:

  *  Build the Maven project to create an artifact (e.g., JAR file).

   * Build the Docker image for the Spring Boot application using the Dockerfile.

   * Use Docker Compose to deploy the Spring Boot and MySQL containers.

   * Test the deployed application to ensure it is running correctly within the Docker containers.

9. Configure the Jenkins pipeline job to use the Jenkinsfile you created.
10. Run the Jenkins pipeline job and monitor the console output for any errors or issues.
