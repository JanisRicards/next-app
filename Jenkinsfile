pipeline {
    environment {
        DOCKER_REGISTRY = "971693106226.dkr.ecr.eu-north-1.amazonaws.com/jenkins-ecr"
        DOCKER_REPO = "jenkins-ecr"
        DOCKER_TAG = "latest"
        AWS_REGION = "eu-north-1"
    }
    agent any
    stages {
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                sh 'docker build -t jenkins-ecr .'
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'a4969178-ffb6-49f1-bb7b-7760e13d9f4f', regionVariable: 'AWS_REGION')]) {
                    sh "aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 971693106226.dkr.ecr.eu-north-1.amazonaws.com"
                    sh "docker tag jenkins-ecr:latest 971693106226.dkr.ecr.eu-north-1.amazonaws.com/jenkins-ecr:latest"
                    sh "docker push 971693106226.dkr.ecr.eu-north-1.amazonaws.com/jenkins-ecr:latest"
                }
            }
        }
    }
}
