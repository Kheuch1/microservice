pipeline {
    agent any

    environment {
        DOCKER_CRED_ID = 'dockerhub-credentials'
        DOCKER_USER = 'cheikhaa'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "--> Utilisation du workspace courant..."
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "--> Construction Frontend..."
                    docker.build(
                        "${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG}",
                        "./frontend"
                    )

                    echo "--> Construction Backend..."
                    docker.build(
                        "${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG}",
                        "./backend"
                    )

                    echo "--> Construction MySQL..."
                    docker.build(
                        "${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG}",
                        "./database"
                    )
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    echo "--> Connexion à Docker Hub..."

                    withCredentials([
                        usernamePassword(
                            credentialsId: env.DOCKER_CRED_ID,
                            usernameVariable: 'DOCKER_USER_NAME',
                            passwordVariable: 'DOCKER_USER_PASS'
                        )
                    ]) {
                        try {
                            bat '''
                                @echo off
                                powershell -NoProfile -NonInteractive -Command "$env:DOCKER_USER_PASS | docker login --username $env:DOCKER_USER_NAME --password-stdin"
                                if errorlevel 1 exit /b 1
                            '''

                            echo "--> Publication des images versionnées..."

                            bat "docker push ${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG}"
                            bat "docker push ${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG}"
                            bat "docker push ${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG}"

                        } finally {
                            echo "--> Déconnexion de Docker Hub..."
                            bat 'docker logout || exit /b 0'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "--> Nettoyage des images locales..."

            bat "docker rmi ${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG} 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG} 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG} 2>nul || exit /b 0"
        }

        success {
            echo "--> SUCCESS : Images versionnées publiées sur Docker Hub !"
        }

        failure {
            echo "--> FAILURE : Le pipeline a rencontré une erreur."
        }
    }
}