
pipeline {
    # Definition du label de l'agent Jenkins
    agent { label 'bijouterie' }

    # Definition des options du pipeline
    options {
        skipDefaultCheckout()   # Permet de ne pas effectuer le checkout par défaut du code source
    }

    # Definition des variables d'environnement
    environment {
        FRONTEND_IMAGE = 'fallougaye/bijouterie-frontend'
        BACKEND_IMAGE  = 'fallougaye/bijouterie-backend'
        MYSQL_IMAGE    = 'fallougaye/bijouterie-mysql'
    }

    stages {
        # Recuperation du code source depuis GitHub
        stage('Checkout') {
            steps {
                echo 'Recuperation du code source depuis GitHub'
                checkout scm
            }
        }

        stage('Verification') {
            steps {
                bat 'docker version'
                bat 'git --version'
            }
        }

        stage('Build') {
            steps {
                echo 'Construction des trois images Docker'
                # Construction des images Docker pour le frontend, le backend et la base de données MySQL
                bat 'docker build -t bijouterie-frontend:build .\\frontend'
                bat 'docker build -t bijouterie-backend:build .\\backend'
                bat 'docker build -t bijouterie-mysql:build .\\database'
            }
        }

        stage('Tag') {
            steps {
                echo "Attribution du tag ${BUILD_NUMBER}"
                # Attribution du tag de build aux images Docker construites
                bat 'docker tag bijouterie-frontend:build %FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker tag bijouterie-backend:build %BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker tag bijouterie-mysql:build %MYSQL_IMAGE%:%BUILD_NUMBER%'
            }
        }

        stage('Login Docker Hub') {
            steps {
                # Connexion à Docker Hub avec les identifiants stockés dans Jenkins
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub', 
                        usernameVariable: 'DOCKERHUB_USER', 
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    # Connexion à Docker Hub avec les identifiants stockés dans Jenkins
                    bat 'echo %DOCKERHUB_TOKEN% | docker login -u %DOCKERHUB_USER% --password-stdin'
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Publication des images sur Docker Hub'
                # Publication des images Docker sur Docker Hub avec le tag de build
                bat 'docker push %FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %MYSQL_IMAGE%:%BUILD_NUMBER%'
            }
        }
    }
}
