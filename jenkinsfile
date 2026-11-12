pipeline {
    agent { label 'bijouterie' }
    stages {
        stage('Test') {
            steps {
                bat 'docker version'
                bat 'git --version'
            }
        }
    }
}