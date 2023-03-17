pipeline {
    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
    }
    environment {
        DOCKER_REGISTRY = "${env.DOCKER_REGISTRY_URL}"
        DOCKER_REPO = "${env.ECR_REPO}"
        AWS_REGION = "${env.AWS_REG}"
    }
    agent any
    stages {
        stage('Login to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', regionVariable: 'AWS_REGION')]) {
                    sh "aws ecr get-login-password --region $AWS_REGION | sudo docker login --username AWS --password-stdin $DOCKER_REGISTRY"
                }
            }
        }
        stage('Pull Docker image') {
            steps {
                script {
                    sh "sudo docker pull $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                }
            }
        }
        stage('Deploy Docker image to ECR') {
            steps {
                script {
                    sh "sudo docker tag $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG $DOCKER_REPO:$DOCKER_TAG"
                    sh "sudo docker push $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                }
            }
        }
    }
}
