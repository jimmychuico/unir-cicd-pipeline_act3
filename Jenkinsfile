pipeline {
    agent {
        label 'docker'
    }
    stages {
        stage('Source') {
            steps {
                git 'https://github.com/jimmychuico/unir-cicd-pipeline_act3.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Building stage!'
                bat 'make build'
            }
        }
        stage('Unit tests') {
            steps {
                bat 'mkdir results'
                bat 'make test-unit'
                archiveArtifacts artifacts: 'results/*.xml'
            }
        }
    }
    post {
        always {
            junit 'results/*_result.xml'
            cleanWs()
        }
    }
}