pipeline {
    agent any

    environment {
        DOCKER_CRED_ID = 'dockerhub-credentials'
        DOCKER_USER    = 'cheikhaa'
        BUILD_TAG      = "${env.BUILD_NUMBER}"
    }

    stages {
        // 1. Récupération du code source (dans le workspace Jenkins)
        stage('Checkout') {
            steps {
                echo "--> Récupération du code..."
                checkout scm
            }
        }

        // 2. Construction des images Docker avec 'def' pour éviter les avertissements
        stage('Build') {
            steps {
                script {
                    echo "--> Construction de l'image Frontend..."
                    def dockerFrontend = docker.build("${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG}", "./frontend")

                    echo "--> Construction de l'image Backend..."
                    def dockerBackend = docker.build("${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG}", "./backend")

                    echo "--> Construction de l'image MySQL..."
                    def dockerMysql = docker.build("${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG}", "./database")
                }
            }
        }

        // 3. Application du double tagging
        stage('Tag') {
            steps {
                script {
                    echo "--> Application du tag 'latest'..."
                    // Note : comme les variables locales s'arrêtent à la fin du stage Build, 
                    // on tague directement via la ligne de commande docker tag sous Windows pour plus de simplicité
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG} ${env.DOCKER_USER}/bijouterie-frontend:latest"
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG} ${env.DOCKER_USER}/bijouterie-backend:latest"
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG} ${env.DOCKER_USER}/bijouterie-mysql:latest"
                }
            }
        }

        // 4. Publication sur le registre Docker Hub
        stage('Push') {
            steps {
                script {
                    echo "--> Connexion et publication sur Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', env.DOCKER_CRED_ID) {
                        // Push des tags de build
                        bat "docker push ${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG}"
                        bat "docker push ${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG}"
                        bat "docker push ${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG}"

                        // Push des tags latest
                        bat "docker push ${env.DOCKER_USER}/bijouterie-frontend:latest"
                        bat "docker push ${env.DOCKER_USER}/bijouterie-backend:latest"
                        bat "docker push ${env.DOCKER_USER}/bijouterie-mysql:latest"
                    }
                }
            }
        }
    }

    // Nettoyage et logs post-exécution (sécurisé pour Windows)
    post {
        always {
            echo "--> Nettoyage des images locales du runner Jenkins..."
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG} 2>nul || exit 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG} 2>nul || exit 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG} 2>nul || exit 0"
        }
        success {
            echo "--> Pipeline CI exécuté avec succès et images publiées sur Docker Hub !"
        }
        failure {
            echo "--> Échec du pipeline CI."
        }
    }
}