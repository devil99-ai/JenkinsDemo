pipeline {
  agent any
  environment {
    AWS_DEFAULT_REGION = 'ap-south-1'
    ECR_REPO_NAME = '203123579158.dkr.ecr.ap-south-1.amazonaws.com/angular-docker'
    IMAGE_TAG = "203123579158.dkr.ecr.ap-south-1.amazonaws.com/angular-docker:${env.BUILD_ID}"
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/devil99-ai/JenkinsDemo.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${env.ECR_REPO_NAME}:${env.BUILD_ID}")
        }
      }
    }
    stage('Scan with Trivy') {
      steps {
        script {
          sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL ${dockerImage.id}'
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com", 'ecr:aws-credentials') {
            dockerImage.push()
          }
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        script {
          sh '''
          aws eks --region $AWS_DEFAULT_REGION update-kubeconfig --name $CLUSTER_NAME
          kubectl apply -f k8s/deployment.yaml
          '''
        }
      }
    }
  }
}
