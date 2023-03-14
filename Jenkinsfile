pipeline {
    environment {
        DOCKER_REGISTRY = "${env.DOCKER_REGISTRY_URL}"
        DOCKER_REPO = "${env.ECR_REPO}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        AWS_REGION = "${env.AWS_REG}"
        SONAR_TOKEN = credentials('SonarQubeV2')
    }
    agent any
    stages {
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool('Sonar-Scanner')
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'sudo npm install'
            }
        }
        stage('Build') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                    sh "sudo docker build -t $DOCKER_REPO:$DOCKER_TAG ."
                }
            }
        }
        stage('Testing test') {
            steps {
                sh 'sudo npm run test'
            }
        }
        stage('ESLint') {
            steps {
                sh 'sudo npm run lint'
            }
        }
        stage('Quality gate check') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        timeout(time: 5, unit: 'MINUTES') {
                            def qg = waitForQualityGate()
                            }
                        }
                    }
                }
            }
        }
        stage('Build and push to ECR') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
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
            post {
                always {
                    cleanWs()
                }
            }
        }
    }

