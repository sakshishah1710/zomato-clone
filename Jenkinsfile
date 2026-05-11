pipeline {
    agent any

    tools {
        jdk 'jdk 17'
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
                git branch: 'main',
                url: 'https://github.com/sakshishah1710/zomato-clone.git'
            }
        }

        stage('Install NPM Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t zomato .'
            }
        }

        stage('Tag & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker',
                        url: 'https://index.docker.io/v1/'
                    ) {

                        sh 'docker tag zomato sakshidocker2002/zomato:latest'
                        sh 'docker push sakshidocker2002/zomato:latest'
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker stop zomato || true'
                sh 'docker rm zomato || true'

                sh '''
                docker run -d \
                --name zomato \
                -p 3000:3000 \
                sakshidocker2002/zomato:latest
                '''
            }
        }
    }

    post {
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.result}: ${env.JOB_NAME}",
                body: """
                <html>
                <body>

                    <h2>Jenkins Build Notification</h2>

                    <p><b>Project:</b> ${env.JOB_NAME}</p>

                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>

                    <p><b>Build Status:</b> ${currentBuild.result}</p>

                    <p><b>Build URL:</b> ${env.BUILD_URL}</p>

                </body>
                </html>
                """,
                to: 'shahsakshi1702@gmail.com',
                mimeType: 'text/html'
            )
        }
    }
}