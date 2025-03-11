pipeline {
    agent any

    environment {
        IMAGE_NAME = "nginx-ci-test"
        CONTAINER_NAME = "nginx_test"
        HOST_PORT = "9889"
        MD5_EXPECTED = ""
        TELEGRAM_BOT_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN"
        TELEGRAM_CHAT_ID = "YOUR_CHAT_ID"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/YOUR_REPO.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh 'docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 ${IMAGE_NAME}'
                    sleep 5 // Даем Nginx запуститься
                }
            }
        }

        stage('Check HTTP Response') {
            steps {
                script {
                    def statusCode = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://localhost:${HOST_PORT}", returnStdout: true).trim()
                    if (statusCode != '200') {
                        sendTelegramMessage("Ошибка CI: HTTP-код ${statusCode} (ожидался 200)")
                        error("HTTP response code check failed")
                    }
                }
            }
        }

        stage('Check MD5 Hash') {
            steps {
                script {
                    MD5_EXPECTED = sh(script: "md5sum index.html | awk '{print \$1}'", returnStdout: true).trim()
                    def MD5_ACTUAL = sh(script: "curl -s http://localhost:${HOST_PORT}/index.html | md5sum | awk '{print \$1}'", returnStdout: true).trim()

                    if (MD5_EXPECTED != MD5_ACTUAL) {
                        sendTelegramMessage("Ошибка CI: MD5 суммы не совпадают!")
                        error("MD5 hash check failed")
                    }
                }
            }
        }

    }

    post {
        always {
            script {
                sh 'docker stop ${CONTAINER_NAME} || true'
                sh 'docker rm ${CONTAINER_NAME} || true'
                sh 'docker rmi ${IMAGE_NAME} || true'
            }
        }
        failure {
            sendTelegramMessage("CI упал! Проверьте логи.")
        }
    }
}

def sendTelegramMessage(message) {
    sh "curl -s -X POST https://api.telegram.org/bot${7397473657:AAGHn2gysmSRPEzebRIdca3ZBe-hxpD6Th4}/sendMessage -d chat_id=${-4604045894} -d text=\"${message}\""
}
