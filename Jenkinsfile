pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
  }
  agent any
  environment {
    registryCredential = 'ecr:eu-north-1:f41c8ac0-f1ee-4a07-8bbb-1b014d174bfb'
    appRegistry = '362447113011.dkr.ecr.eu-north-1.amazonaws.com/petclinic-ecr-images'
    awsRegistry = "https://362447113011.dkr.ecr.eu-north-1.amazonaws.com"
    cluster = "ProdCluster"
    service = "petclinic-app"
} 
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
            sh 'sudo docker build  -f ./Dockerfile.multi -t petclinic:${BUILD_NUMBER} .'
        }
    }

    stage('Push Docker image to ECR') { 
        steps {
            script {
                docker.withRegistry(awsRegistry, "ecr:eu-north-1:aws-creds") {
                sh """
                    docker tag petclinic:${BUILD_NUMBER} 362447113011.dkr.ecr.eu-north-1.amazonaws.com/petclinic-ecr-images:${BUILD_NUMBER}
                    docker tag petclinic:${BUILD_NUMBER} 362447113011.dkr.ecr.eu-north-1.amazonaws.com/petclinic-ecr-images:latest
                    docker push --all-tags 362447113011.dkr.ecr.eu-north-1.amazonaws.com/petclinic-ecr-images
                    docker image prune
                """
                }
            }
        }           
    }
    stage('Deploy to ECS staging') {
        steps {
            withAWS(credentials: 'aws-creds', region: 'eu-north-1') {
                sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                } 
            }
        }
    }
}
