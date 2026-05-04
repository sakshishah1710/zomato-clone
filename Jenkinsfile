pipeline {
    agent any

    tools {
        nodejs 'node23'   // keep JDK only if configured properly
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
                git branch: 'main', url: 'https://github.com/sakshishah1710/zomato-clone.git'
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

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                script {
                    withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
    sh 'docker tag zomato sakshidocker2002/zomato:latest'
    sh 'docker push sakshidocker2002/zomato:latest'
}
                    }
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

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                <html>
                <body>
                    <h2>Build Notification</h2>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>URL:</b> ${env.BUILD_URL}</p>
                </body>
                </html>
                """,
                to: 'shahsakshi1702@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy.txt'
        }
    }
}