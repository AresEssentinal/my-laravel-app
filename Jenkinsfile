pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aresessentinal/laravel-app'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                script {
                    sh 'cp .env.production .env'
                    sh 'composer install --no-dev --optimize-autoloader --no-interaction'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'docker compose down --remove-orphans || true'
                    sh 'docker compose up -d --build'
                    
                    sh 'docker compose exec -T app chown -R www:www storage bootstrap/cache || true'
                    sh 'docker compose exec -T app php artisan optimize:clear || true'
                    sh 'docker compose exec -T app php artisan migrate --force || true'
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Laravel успешно развернут!'
            echo 'Приложение: http://localhost:8000'
        }
        failure {
            echo '❌ Ошибка деплоя!'
        }
    }
}
