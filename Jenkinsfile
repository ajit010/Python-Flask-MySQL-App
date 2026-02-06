pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'yourdockerhubuser/flask-app'
        IMAGE_TAG = "${BUILD_NUMBER}"

        MYSQL_ROOT_PASSWORD = credentials('mysql-root-password')
        MYSQL_DATABASE      = credentials('mysql-database')
        MYSQL_USER          = credentials('mysql-user')
        MYSQL_PASSWORD      = credentials('mysql-password')
        FLASK_SECRET_KEY    = credentials('flask-secret-key')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-creds',
                    url: 'https://github.com/your-org/flask-app.git'
            }
        }

        stage('Create .env File') {
            steps {
                sh '''
                cat <<EOF > .env
MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
MYSQL_DATABASE=${MYSQL_DATABASE}
MYSQL_USER=${MYSQL_USER}
MYSQL_PASSWORD=${MYSQL_PASSWORD}
FLASK_SECRET_KEY=${FLASK_SECRET_KEY}
IMAGE_TAG=${IMAGE_TAG}
EOF
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ./flask-app
                '''
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh '''
                docker-compose down || true
                docker rm -f flask-app mysql-db || true
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker-compose up -d --build
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful with image tag: ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
