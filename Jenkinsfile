pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-creds') // Referência às credenciais globais
        IMAGE_NAME = "luandocs/myapp:jenkins" // Substitua pelo seu nome de usuário e nome da imagem
    }

    stages {
        stage('Login to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                        echo "Logado no Docker Hub!"
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def appImage = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                    appImage.push()
                }
            }
        }
    }
}