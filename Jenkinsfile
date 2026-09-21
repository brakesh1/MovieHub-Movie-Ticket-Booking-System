pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    set -e

                    echo "===== Docker ====="
                    docker --version
                    docker compose version

                    echo "===== Repository ====="
                    pwd
                    ls -la
                '''
            }
        }

        stage('Validate Compose') {
            steps {
                sh '''
                    set -e
                    docker compose config --quiet
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -e
                    docker compose build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    docker compose up -d

                    echo "Waiting for services..."
                    sleep 15

                    docker compose ps
                '''
            }
        }

        stage('Verify Database') {
            steps {
                sh '''
                    set -e

                    STATUS=$(docker inspect \
                        --format='{{.State.Health.Status}}' \
                        moviehub-db)

                    echo "Database status: $STATUS"

                    test "$STATUS" = "healthy"
                '''
            }
        }

        stage('Verify Backend') {
            steps {
                sh '''
                    set -e

                    docker compose exec -T backend \
                        python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:8000/').read().decode())"
                '''
            }
        }

        stage('Verify Frontend') {
            steps {
                sh '''
                    set -e

                    curl --fail --retry 10 --retry-delay 3 \
                        http://localhost/

                    echo
                    echo "Frontend health check passed"
                '''
            }
        }
    }

    post {
        success {
            echo 'MovieHub deployment successful.'
        }

        failure {
            echo 'MovieHub deployment failed.'

            sh '''
                docker compose ps || true

                echo "===== BACKEND LOGS ====="
                docker compose logs --tail=30 backend || true

                echo "===== DATABASE LOGS ====="
                docker compose logs --tail=30 db || true

                echo "===== FRONTEND LOGS ====="
                docker compose logs --tail=40 frontend || true
            '''
        }
    }
}
