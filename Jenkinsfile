pipeline {
    environment {
        DOCKER_REGISTRY = "${env.DOCKER_REGISTRY_URL}"
        DOCKER_REPO = "${env.ECR_REPO}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        AWS_REGION = "${env.AWS_REG}"
    }
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                    sh "npm run build"
                    sh "npm run test"
                    sh "npm run lint"
                    sh "sudo docker build -t $DOCKER_REPO:$DOCKER_TAG ."
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Build and push to ECR') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                    sh "npm run build"
                    sh "npm run test"
                    sh "npm run lint"
                    sh "sudo docker build -t $DOCKER_REPO:$DOCKER_TAG ."
                    withCredentials([aws(credentialsId: 'aws-credentials', regionVariable: 'AWS_REGION')]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | sudo docker login --username AWS --password-stdin $DOCKER_REGISTRY"
                        sh "sudo docker tag $DOCKER_REPO:$DOCKER_TAG $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                        sh "sudo docker push $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                    }
                }
            }
        }
        stage('Cleanup workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
    }
}
