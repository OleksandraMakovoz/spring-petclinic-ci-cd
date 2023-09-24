pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
         git(
            url: 'https://github.com/OleksandraMakovoz/spring-petclinic.git',
            branch: 'main'
         )
      }
    }
    stage('Build') {
      steps {
         sh './mvnw compile'
      }
    }
    stage('Tests') {
      steps {
         sh './mvnw test -Dspring.profiles.active=mysql -DskipTests'
      }
    }
    stage('Package. Build Docker image') {
        steps {
            sh 'dockerImage=$(docker build  -f ./Dockerfile.multi -t petclinic:latest .)'
            sh 'echo ${dockerImage}'
        }
    }

    stage('Push image to ECR') {
        steps {
            sh 'docker build  -f ./Dockerfile.multi -t petclinic:latest .'
        }
    }


  }
}
