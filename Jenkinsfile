pipeline {
  agent any

  environment {
    SSH_KEY = credentials('ssh-key')
    webserver = 'ubuntu@3.88.177.192'
  }

  stages {
    stage('Build') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')])
          sh 'docker build -t fastapi-app:latest .'
          sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
          sh 'docker push 3rigs/work:latest'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sshagent(credentials: [SSH_KEY]) {
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker pull 3rigs/work:latest"'
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker stop fastapi-app"'
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker rm fastapi-app"'
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker run -d --name fastapi-app -p 8000:8000 3rigs/work:latest"'
          }
        }
      }
    }

    stage('Monitoring Setup') {
      steps {
        script {
          sshagent(credentials: [SSH_KEY]) {
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker run -d --name prometheus -p 9090:9090 prom/prometheus"'
            sh 'ssh -o StrictHostKeyChecking=no ${webserver} "docker run -d --name grafana -p 3000:3000 grafana/grafana"'
          }
        }
      }
    }
  }
}
