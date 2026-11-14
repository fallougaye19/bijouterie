pipeline {
    // Definition du label de l'agent Jenkins
    agent { label 'bijouterie' }
    // Definition des options du pipeline
    options {
        skipDefaultCheckout()
    }
    // Definition des variables d'environnement
    environment {
        FRONTEND_IMAGE = 'fallougaye/bijouterie-frontend'
        BACKEND_IMAGE  = 'fallougaye/bijouterie-backend'
        MYSQL_IMAGE    = 'fallougaye/bijouterie-mysql'
    }
    stages {
        // Recuperation du code source depuis GitHub
        stage('Checkout') {
            steps {
                echo 'Recuperation du code source depuis GitHub'
                checkout scm
            }
        }
        // Verification de Docker et Git
        stage('Verification') {
            steps {
                bat 'docker version'
                bat 'git --version'
            }
        }
        // Construction des trois images Docker
        stage('Build') {
            steps {
                echo 'Construction des trois images Docker'
                bat 'docker build -t bijouterie-frontend:build .\\frontend'
                bat 'docker build -t bijouterie-backend:build .\\backend'
                bat 'docker build -t bijouterie-mysql:build .\\database'
            }
        }
        // Attribution du numero de build aux images
        stage('Tag') {
            steps {
                echo "Attribution du tag ${BUILD_NUMBER}"
                bat 'docker tag bijouterie-frontend:build %FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker tag bijouterie-backend:build %BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker tag bijouterie-mysql:build %MYSQL_IMAGE%:%BUILD_NUMBER%'
            }
        }
        // Publication des images sur Docker Hub
        stage('Push') {
            steps {
                echo 'Publication des images sur Docker Hub'
                bat 'docker push %FRONTEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %BACKEND_IMAGE%:%BUILD_NUMBER%'
                bat 'docker push %MYSQL_IMAGE%:%BUILD_NUMBER%'
            }
        }
        // Mise a jour automatique des manifests Kubernetes
        stage('Update manifests') {
            steps {
                script {
                    echo "Mise a jour des manifests avec le tag ${BUILD_NUMBER}"
                    def cibles = [
                        'k8s/06-backend-deployment.yaml'   : env.BACKEND_IMAGE,
                        'k8s/09-frontend-deployment.yaml'  : env.FRONTEND_IMAGE,
                        'k8s/02-mysql-deployment.yaml'     : env.MYSQL_IMAGE
                    ]
                    cibles.each { fichier, image ->
                        def contenu = readFile(file: fichier, encoding: 'UTF-8')
                        def motif = "(?m)(image:\\s*${java.util.regex.Pattern.quote(image)}:)[^\\r\\n]+"
                        def remplacement = '$1' + env.BUILD_NUMBER
                        def nouveauContenu = contenu.replaceAll(motif, remplacement)
                        if (nouveauContenu == contenu) {
                            error("Aucune image correspondante trouvee dans ${fichier}")
                        }
                        if (!nouveauContenu.contains("image: ${image}:${env.BUILD_NUMBER}")) {
                            error("Le remplacement de l'image a echoue dans ${fichier}")
                        }
                        writeFile(file: fichier, text: nouveauContenu, encoding: 'UTF-8')
                        echo "Manifest mis a jour : ${fichier}"
                    }
                }
                // Configuration de l'auteur du commit
                bat 'git config user.name "Jenkins"'
                bat 'git config user.email "jenkins@users.noreply.github.com"'
                // Ajout des trois fichiers modifies
                bat 'git add k8s/06-backend-deployment.yaml k8s/09-frontend-deployment.yaml k8s/02-mysql-deployment.yaml'
                // Verification des modifications
                script {
                    def changements = bat(script: 'git diff --cached --quiet', returnStatus: true)
                    if (changements == 0) {
                        echo 'Aucune modification a committer.'
                    } else {
                        // Creation du commit
                        bat 'git commit -m "ci: mise a jour des images - build %BUILD_NUMBER% [skip ci]"'
                        // Publication sur GitHub
                        withCredentials([gitUsernamePassword(credentialsId: 'github-credentials', gitToolName: 'Default')]) {
                            bat 'git push origin HEAD:main'
                        }
                    }
                }
            }
        }
    }
}
