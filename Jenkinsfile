pipeline {
    agent any
    
    stages {
        stage('build') {
            agent { docker { image 'node' } }
            steps {
                sh 'node --version'
            }
        }
    }
}