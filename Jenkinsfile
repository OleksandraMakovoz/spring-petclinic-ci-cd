pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
  }
  agent any
  environment {
    registryCredential = 'ecr:eu-north-1:aws-creds'
    appRegistry = '362447113011.dkr.ecr.eu-north-1.amazonaws.com/petclinic-ecr-images'
    awsRegistry = 'https://362447113011.dkr.ecr.eu-north-1.amazonaws.com'
    cluster = 'ProdCluster'
    service = 'petclinic-app'
  }
  stages {
    stage('Checkout Terraform') {
      steps {
        git(
          url: 'https://github.com/OleksandraMakovoz/spring-petclinic-iac.git',
          branch: 'main'
        )
      }
    }

    stage('Terraform init&plan') {
      steps {
        withAWS(credentials: 'aws-creds', region: 'eu-north-1') {
          sh 'terraform init'
          sh 'terraform plan'
        }
      }
    }

    stage('Terraform apply') {
      steps {
        withAWS(credentials: 'aws-creds', region: 'eu-north-1') {
          sh 'terraform apply -auto-approve'
          sh 'rm -r .terraform'
        }
      }
    }

    stage('Cloning Petclinic App Repo') {
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
        sh './mvnw test'
      }
    }
    stage('Package. Build Docker image') {
        steps {
            sh 'docker build  -f ./Dockerfile.multi -t petclinic:${BUILD_NUMBER} .'
        }
    }

    stage('Push Docker image to ECR') {
        steps {
            script {
              docker.withRegistry(awsRegistry, registryCredential) {
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
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment --desired-count 1'
          sh 'aws ecs list-tasks --cluster ${cluster}'
            }
        }
    }
  }
}
