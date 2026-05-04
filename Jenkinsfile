pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git 'https://github.com/sakshishah1710/zomato-clone.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs . > trivy.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t zomato .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withDockerRegistry(credentialsId: 'docker') {
                    sh 'docker tag zomato sakshishah/zomato:latest'
                    sh 'docker push sakshishah/zomato:latest'
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker stop zomato || true'
                sh 'docker rm zomato || true'
                sh 'docker run -d -p 3000:3000 --name zomato sakshishah/zomato:latest'
            }
        }
    }
}