pipeline {
    agent any

    environment {
        DOCKER_CRED_ID = 'dockerhub-credentials'
        DOCKER_USER    = 'cheikhaa'
        BUILD_TAG      = "${env.BUILD_NUMBER}"
    }

    stages {
        // 1. Récupération du code source depuis Git
        stage('Checkout') {
            steps {
                echo "--> Récupération du code depuis le dépôt Git..."
                checkout scm
            }
        }

        // 2. Construction des images Docker pour chaque microservice
        stage('Build') {
            steps {
                script {
                    echo "--> Construction de l'image Frontend..."
                    dockerFrontend = docker.build("${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG}", "./frontend")

                    echo "--> Construction de l'image Backend..."
                    dockerBackend = docker.build("${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG}", "./backend")

                    echo "--> Construction de l'image MySQL..."
                    dockerMysql = docker.build("${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG}", "./database")
                }
            }
        }

        // 3. Application du double tagging (Numéro de build + latest)
        stage('Tag') {
            steps {
                script {
                    echo "--> Application du tag 'latest'..."
                    dockerFrontend.tag("latest")
                    dockerBackend.tag("latest")
                    dockerMysql.tag("latest")
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
                        dockerFrontend.push("${env.BUILD_TAG}")
                        dockerBackend.push("${env.BUILD_TAG}")
                        dockerMysql.push("${env.BUILD_TAG}")

                        // Push des tags latest
                        dockerFrontend.push("latest")
                        dockerBackend.push("latest")
                        dockerMysql.push("latest")
                    }
                }
            }
        }
    }

    // Nettoyage et logs post-exécution
    post {
        always {
            echo "--> Nettoyage des images locales du runner Jenkins..."
            sh "docker rmi ${env.DOCKER_USER}/bijouterie-frontend:${env.BUILD_TAG} || true"
            sh "docker rmi ${env.DOCKER_USER}/bijouterie-backend:${env.BUILD_TAG} || true"
            sh "docker rmi ${env.DOCKER_USER}/bijouterie-mysql:${env.BUILD_TAG} || true"
        }
        success {
            echo "--> Pipeline CI exécuté avec succès et images publiées !"
        }
        failure {
            echo "--> Échec du pipeline CI."
        }
    }
}