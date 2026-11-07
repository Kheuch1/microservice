pipeline {
    agent any

    environment {
        DOCKER_CRED_ID = 'dockerhub-credentials'
        DOCKER_USER    = 'cheikhaa'
        BUILD_TAG      = "${env.BUILD_NUMBER}"
    }

    stages {
        // 1. Étape de Checkout simplifiée pour un script Inline
        stage('Checkout') {
            steps {
                echo "--> Utilisation du workspace courant..."
                // Pas besoin de 'checkout scm' ici puisque vous êtes en mode script Inline 
                // et que Jenkins positionne déjà les fichiers dans le workspace.
            }
        }

        // 2. Construction des images Docker pour chaque microservice
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

        // 3. Application du double tagging (Numéro de build + latest)
        stage('Tag') {
            steps {
                script {
                    echo "--> Application du tag 'latest'..."
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
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CRED_ID}", 
                                                      usernameVariable: 'DOCKER_USER_NAME', 
                                                      passwordVariable: 'DOCKER_USER_PASS')]) {
                        
                        // Connexion sécurisée à Docker Hub avec votre token
                        bat "echo %DOCKER_USER_PASS% | docker login -u %DOCKER_USER_NAME% --password-stdin"

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

    // Nettoyage des images locales sur le runner après exécution
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