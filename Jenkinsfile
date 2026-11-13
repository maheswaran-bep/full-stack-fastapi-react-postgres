
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
                withCredentials([
                    string(credentialsId: 'postgres-password',
                           variable: 'POSTGRES_PASSWORD')
                ]) {
                    sh 'docker compose config --quiet'
                }
            }
        }

        stage('Build Images') {
            steps {
                withCredentials([
                    string(credentialsId: 'postgres-password',
                           variable: 'POSTGRES_PASSWORD')
                ]) {
                    sh '''
                        docker compose build \
                          --progress=plain
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    string(credentialsId: 'postgres-password',
                           variable: 'POSTGRES_PASSWORD')
                ]) {
                    sh 'docker compose up -d'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker compose ps

                    docker inspect notes-postgres \
                      --format='PostgreSQL: {{.State.Status}}'

                    docker inspect notes-backend \
                      --format='Backend: {{.State.Status}}'

                    docker inspect notes-frontend \
                      --format='Frontend: {{.State.Status}}'

                    docker logs --tail=30 notes-backend

                    docker logs --tail=30 notes-frontend
                '''
            }
        }
    }

    post {
        success {
            echo 'Full Stack application deployed successfully!'
            echo 'Frontend: http://localhost:3025'
            echo 'Backend:  http://localhost:8025/docs'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }

        always {
            sh 'docker compose ps || true'
        }
    }
}
