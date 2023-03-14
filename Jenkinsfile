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
                    def scannerHome = tool('sq1')
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh ' npm install'
            }
        }
        stage('Build') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/JanisRicards/next-app.git'
                    sh " docker build -t $DOCKER_REPO:$DOCKER_TAG ."
                }
            }
        }
        stage('Testing test') {
            steps {
                sh 'npm run test'
            }
        }
        stage('ESLint') {
            steps {
                sh 'npm run lint'
            }
        }
        stage('Quality gate check') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sleep(60)
                        timeout(time: 5, unit: 'MINUTES') {
                            def qg = waitForQualityGate()
                            print "Finished waiting"
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
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
                    sh " docker build -t $DOCKER_REPO:$DOCKER_TAG ."
                    withCredentials([aws(credentialsId: 'aws-credentials', regionVariable: 'AWS_REGION')]) {
                        sh "aws ecr get-login-password --region $AWS_REGION |  docker login --username AWS --password-stdin $DOCKER_REGISTRY"
                        sh " docker tag $DOCKER_REPO:$DOCKER_TAG $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
                        sh " docker push $DOCKER_REGISTRY/$DOCKER_REPO:$DOCKER_TAG"
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
}
