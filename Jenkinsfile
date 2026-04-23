pipeline {
  agent any

  stages {
    stage('Clone') {
      steps {
        git 'https://github.com/sakshishah1710/zomato-clone.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t zomato-app ./backend'
      }
    }

    stage('Stop Old Container') {
      steps {
        sh 'docker stop zomato-container || true'
        sh 'docker rm zomato-container || true'
      }
    }

    stage('Run Container') {
      steps {
        sh 'docker run -d -p 5000:5000 --name zomato-container zomato-app'
      }
    }
  }
}