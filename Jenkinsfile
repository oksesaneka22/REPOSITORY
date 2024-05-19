pipeline {
    agent any

    environment {
        // Додаємо креденшіали для Docker
        DOCKER_CREDENTIALS_ID = '20d20db0-e450-4938-adde-23e1930a571a'
        CONTAINER_NAME = 'oksesaneka22/site2223'
    }


    stages {


       stage('Вхід у Docker') {
            steps {
                script {
                    // Використовуємо креденшіали з Jenkins для входу в Docker
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Білд Docker зображення') {
            steps {
                script {
                    // Будуємо Docker зображення
                    sh 'docker build -t kuzma343/kuzma_branch:version${BUILD_NUMBER} .'
                }
            }
        }

          stage('Тегування Docker зображення') {
            steps {
                script {
                    // Додаємо тег 'latest' до збудованого образу
                    sh 'docker tag kuzma343/kuzma_branch:version${BUILD_NUMBER} kuzma343/kuzma_branch:latest'
                }
            }
        }

        stage('Пуш у Docker Hub') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker push kuzma343/kuzma_branch:version${BUILD_NUMBER}'
                    sh 'docker push kuzma343/kuzma_branch:latest'
                }
            }
        }

        stage('Зупинка та видалення старого контейнера') {
            steps {
                script {
                    // Спроба зупинити та видалити старий контейнер, якщо він існує
                    sh """
                    if [ \$(docker ps -aq -f name=^${CONTAINER_NAME}\$) ]; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    else
                        echo "Контейнер ${CONTAINER_NAME} не знайдено. Продовжуємо..."
                    fi
                    """
                }
            }
        }

             stage('Чистка старих образів') {
            steps {
                script {
                    // Пушимо зображення на Docker Hub
                    sh 'docker image prune -a --filter "until=24h" --force'

                }
            }
        }


        stage('Запуск Docker контейнера') {
            steps {
                script {
                    // Запускаємо Docker контейнер з новим зображенням
                    sh 'docker run -d -p 8081:80 --name ${CONTAINER_NAME} --health-cmd="curl --fail http://localhost:80 || exit 1" kuzma343/kuzma_branch:version${BUILD_NUMBER}'

                }
            }
        }
    }
}
