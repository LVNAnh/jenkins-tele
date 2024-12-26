pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lvnanh/jenkins-tele'
        DOCKER_TAG = 'latest'
        TELEGRAM_BOT_TOKEN = '7577054664:AAEdeS1fagGs9B9YwBsCT63k3aSFqKwBlkU'
        TELEGRAM_CHAT_ID = '5333038188'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/LVNAnh/jenkins-tele.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image for linux/amd64 platform...'
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", "--platform linux/amd64 .")
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy Golang to DEV') {
            steps {
                script {
                    echo 'Clearing jenkins-tele-related images and containers...'
                    sh '''
                        docker container stop jenkins-tele || echo "No container named jenkins-tele to stop"
                        docker container rm jenkins-tele || echo "No container named jenkins-tele to remove"
                        docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG} || echo "No image ${DOCKER_IMAGE}:${DOCKER_TAG} to remove"
                    '''

                    echo 'Deploying to DEV environment...'
                    sh '''
                        docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}

                        # Kiểm tra và tạo network nếu cần
                        docker network inspect dev >/dev/null 2>&1 || docker network create dev

                        docker container run -d --rm --name jenkins-tele -p 3000:3000 --network dev ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                def message = "✅ Jenkins Build SUCCESS: Job '${env.JOB_NAME}' Build #${env.BUILD_NUMBER}. Check: ${env.BUILD_URL}"
                sh """
                curl -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=${message}"
                """
            }
        }
        failure {
            script {
                def message = "❌ Jenkins Build FAILURE: Job '${env.JOB_NAME}' Build #${env.BUILD_NUMBER}. Check: ${env.BUILD_URL}"
                sh """
                curl -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=${message}"
                """
            }
        }
        always {
            cleanWs()
        }
    }
}
