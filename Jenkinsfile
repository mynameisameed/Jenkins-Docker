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

    
