pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'Java17'
    }

    stages {
        stage('Checkout') {
            steps {
                 sh 'echo passed'
                //git 'https://github.com/your-username/simple-java-webapp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

         stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://107.23.200.12:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=MyProject \
                        -Dsonar.host.url=$SONAR_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }
        /*
        stage('Dependency-Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
                    sh '''
                    /opt/dependency-check/bin/dependency-check.sh \
                    --project simple-java-webapp \
                    --scan . \
                    --out dependency-check-report \
                    --nvdApiKey=$NVD_KEY
                    '''
                }
            }
        }
        */

        stage('Build Docker Image') {
            steps {
                // Clean up old container/image if any
                sh '''
                    docker rm -f webapp || true
                    docker rmi -f simple-java-webapp:latest || true
                    docker build -t simple-java-webapp:latest .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image --scanners vuln \
                    --cache-dir /var/lib/jenkins/.trivy-cache \
                    --severity HIGH,CRITICAL \
                    simple-java-webapp:latest
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d --name webapp -p 9090:9090 simple-java-webapp:latest'
            }
        }
    }
}
