
pipeline {
    agent any

    environment {
        APP_DIR = '/home/ubuntu/full-stack-fastapi-react-postgres'
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                    set -e

                    echo "Checking Docker..."
                    docker --version
                    docker compose version

                    echo "Checking project directory..."
                    test -d "$APP_DIR"

                    cd "$APP_DIR"

                    echo "Checking Docker Compose configuration..."
                    docker compose config --quiet

                    echo "Environment check successful!"
                '''
            }
        }

        stage('Get Latest Code') {
            steps {
                sh '''
                    set -e
                    cd "$APP_DIR"

                    # Pull updates if this repository has an upstream branch.
                    if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
                        if git rev-parse --abbrev-ref \
                            --symbolic-full-name '@{u}' >/dev/null 2>&1; then

                            echo "Pulling latest code..."
                            git pull --ff-only

                        else
                            echo "No Git upstream configured."
                            echo "Using existing project files."
                        fi
                    else
                        echo "Git repository not found."
                        echo "Using existing project files."
                    fi
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    set -e
                    cd "$APP_DIR"

                    echo "Building backend and frontend images..."

                    docker compose build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -e
                    cd "$APP_DIR"

                    echo "Starting application..."

                    docker compose up -d --remove-orphans

                    echo "Application deployment completed!"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -e
                    cd "$APP_DIR"

                    echo "Checking running containers..."

                    docker compose ps

                    echo "Checking backend endpoint..."

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:8000/docs >/dev/null; then
                            echo "Backend is responding!"
                            break
                        fi

                        if [ "$i" -eq 15 ]; then
                            echo "Backend did not respond."
                            docker compose logs --tail=100 backend
                            exit 1
                        fi

                        sleep 3
                    done

                    echo "Checking frontend..."

                    curl -f http://127.0.0.1/ >/dev/null

                    echo "Frontend is responding!"
                    echo "Deployment verified successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD SUCCESS: Application deployed successfully!'
        }

        failure {
            echo 'CI/CD FAILED: Check the Jenkins console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
