pipeline {
    environment {
        DOCKER_REGISTRY = "971693106226.dkr.ecr.eu-central-1.amazonaws.com/jenkins-ecr"
        DOCKER_REPO = "jenkins-ecr"
        DOCKER_TAG = "latest"
        AWS_REGION = "eu-central-1"
    }
    agent any
    stages {
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                sh 'docker build -t $DOCKER_REPO:$DOCKER_TAG .'
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.DOCKER_REGISTRY}"
                    sh "docker tag $DOCKER_REPO:$DOCKER_TAG $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                    sh "docker push $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                }
            }
        }
    }
}
