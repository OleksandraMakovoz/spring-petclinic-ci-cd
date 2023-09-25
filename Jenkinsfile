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
    // stage('Build') {
    //   steps {
    //      sh './mvnw compile'
    //   }
    // }
    // stage('Tests') {
    //   steps {
    //      sh './mvnw test -Dspring.profiles.active=mysql -DskipTests'
    //   }
    // }
    // stage('Package. Build Docker image') {
    //     steps {
    //         // sh 'docker build  -f ./Dockerfile.multi -t petclinic:latest .'
    //     }
    // }

    stage('Build image') { 
        steps {
            script {
                docker.withRegistry("https://362447113011.dkr.ecr.eu-north-1.amazonaws.com", "ecr:eu-north-1:f41c8ac0-f1ee-4a07-8bbb-1b014d174bfb") {
                def customImage = docker.build("my-petclinic:latest", "-f ./Dockerfile.multi")
                docker.image(customImage).push()
                }
            }
        }           
    }
    }
}
