pipeline {
    environment {
        DOCKER_REGISTRY = "${env.DOCKER_REGISTRY_URL}"
        DOCKER_REPO = "${env.ECR_REPO}"
        AWS_REGION = "${env.AWS_REG}"
        IMAGE_TAG = "${params.IMAGE_TAG}"
    }
    agent any
    stages {
        stage('Fetch Image Tag') {
            steps {
                script {
                    IMAGE_TAG = sh(script: 'echo $IMAGE_TAG', returnStdout: true).trim()
                }
            }
        }
        stage('Pull Image from ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'aws-credentials', regionVariable: 'AWS_REGION')]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | sudo docker login --username AWS --password-stdin $DOCKER_REGISTRY"
                        sh "sudo docker pull $DOCKER_REGISTRY/$DOCKER_REPO:$IMAGE_TAG"
                    }
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'aws-credentials', regionVariable: 'AWS_REGION')]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | sudo docker login --username AWS --password-stdin $DOCKER_REGISTRY"
                        sh "sudo docker tag $DOCKER_REGISTRY/$DOCKER_REPO:$IMAGE_TAG $DOCKER_REGISTRY/$DOCKER_REPO:latest"
                        sh "aws ecs update-service --cluster <cluster-name> --service <service-name> --force-new-deployment --image $DOCKER_REGISTRY/$DOCKER_REPO:latest"
                    }
                }
            }
        }
    }
}
