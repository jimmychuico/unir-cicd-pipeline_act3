pipeline {
    agent { label 'docker' }

    stages {
        stage('Source') {
            steps {
                git 'https://github.com/jimmychuico/unir-cicd-pipeline_act3.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker images...'
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

        stage('API tests') {
            steps {
                bat 'mkdir results'
                bat 'make test-api'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/*'
                    junit 'results/*api_result.xml'
                }
            }
        }

        stage('E2E tests') {
            steps {
                bat 'mkdir results'
                bat 'make test-e2e'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/**'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
