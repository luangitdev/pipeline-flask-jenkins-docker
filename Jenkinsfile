pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-creds') // Referência às credenciais globais
        IMAGE_NAME = "luandocs/myapp-jenkins" // Substitua pelo seu nome de usuário e nome da imagem
    }

    // Gatilhos para executar o pipeline.
    triggers {
        pollSCM('H/1 * * * *') // Verifica o repositório a cada 1 minuto.
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/luangitdev/pipeline-flask-jenkins-docker'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // docker-hub-creds é a credencial de acesso do docker hub dentro do Jenkins
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
                    appImage.push('latest') // Atualiza a versão mais recente
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executado com sucesso!"
        }
        failure {
            echo "Ocorreu um erro no pipeline."
        }
    }
}