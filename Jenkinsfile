
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy Application') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose ps
                    docker logs --tail=50 notes-backend
                    docker logs --tail=50 notes-frontend
                '''
            }
        }
    }

    post {
        success {
            echo 'Full Stack application deployed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
        always {
            sh 'docker compose ps || true'
        }
    }
}
