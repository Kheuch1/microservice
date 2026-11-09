// Jenkinsfile déclaratif : pipeline CI de la Bijouterie (Jenkins sous Windows)
pipeline {

    // Le pipeline s'exécute sur le Jenkins local (Windows) : il doit pouvoir lancer Docker Desktop.
    agent any

    // Variables réutilisées dans tous les stages
    environment {
        DOCKER_CRED_ID = 'dockerhub-credentials'     // identifiant du credential Jenkins contenant le token Docker Hub
        DOCKER_USER    = 'cheikhaa'                // compte Docker Hub
        IMAGE_TAG      = "${BUILD_NUMBER}"         // tag de version des images : le numéro de build Jenkins (traçabilité)
    }

    // Déclencheur : Jenkins interroge Git toutes les 2 minutes environ
    // (un webhook GitHub ne peut pas joindre un Jenkins local sans tunnel)
    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        // ---------- STAGE 1 : Checkout ----------
        // Récupère le code du dépôt configuré dans le job
        stage('Checkout') {
            steps {
                echo "--> Récupération du code depuis Git..."
                checkout scm
            }
        }

        // ---------- STAGE 2 : Build ----------
        // Construit les 3 images, chacune à partir du Dockerfile de son dossier.
        // Nom de l'image : <user>/<service>:<numéro de build>
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

        // ---------- STAGE 3 : Tag ----------
        // Double tag : le numéro de build existe déjà (traçabilité, utilisé dans k8s/),
        // "latest" comme raccourci vers la dernière version construite.
        stage('Tag') {
            steps {
                script {
                    echo "--> Ajout du tag latest..."
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG} ${env.DOCKER_USER}/bijouterie-frontend:latest"
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG} ${env.DOCKER_USER}/bijouterie-backend:latest"
                    bat "docker tag ${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG} ${env.DOCKER_USER}/bijouterie-mysql:latest"
                }
            }
        }

        // ---------- STAGE 4 : Push ----------
        // Connexion à Docker Hub avec le credential Jenkins,
        // puis envoi des 3 images avec leurs 2 tags.
        // retry(3) : en cas de coupure réseau (timeout), Jenkins retente jusqu'à 3 fois.
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
                            // Le token est lu depuis une variable d'environnement et passé
                            // via stdin : il n'apparaît ni dans la commande ni dans les logs
                            bat '''
                                @echo off
                                powershell -NoProfile -NonInteractive -Command "$env:DOCKER_USER_PASS | docker login --username $env:DOCKER_USER_NAME --password-stdin"
                                if errorlevel 1 exit /b 1
                            '''

                            echo "--> Publication des images versionnées..."
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG}"
                            }
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG}"
                            }
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG}"
                            }

                            echo "--> Publication des tags latest..."
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-frontend:latest"
                            }
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-backend:latest"
                            }
                            retry(3) {
                                bat "docker push ${env.DOCKER_USER}/bijouterie-mysql:latest"
                            }

                        } finally {
                            // Exécuté même en cas d'erreur : on ferme toujours la session
                            echo "--> Déconnexion de Docker Hub..."
                            bat 'docker logout || exit /b 0'
                        }
                    }
                }
            }
        }
    }

    // Exécuté à la fin du pipeline
    post {
        always {
            echo "--> Nettoyage des images locales..."

            // "|| exit /b 0" : on ignore l'erreur si l'image n'existe déjà plus
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-frontend:${env.IMAGE_TAG} 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-backend:${env.IMAGE_TAG} 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-mysql:${env.IMAGE_TAG} 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-frontend:latest 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-backend:latest 2>nul || exit /b 0"
            bat "docker rmi ${env.DOCKER_USER}/bijouterie-mysql:latest 2>nul || exit /b 0"
        }

        success {
            echo "--> SUCCESS : Images versionnées publiées sur Docker Hub !"
        }

        failure {
            echo "--> FAILURE : Le pipeline a rencontré une erreur."
        }
    }
}