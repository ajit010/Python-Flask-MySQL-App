pipeline {
    agent any
    environment {
        DEPLOY_HOST = 'ec2-2-public-ip'
        DEPLOY_USER = 'ubuntu'

        MYSQL_ROOT_PASSWORD = credentials('mysql-root-password')
        MYSQL_DATABASE = credentials('mysql-database')
        MYSQL_USER = credentials('mysql-user')
        MYSQL_PASSWORD = credentials('mysql-password')

        FLASK_SECRET_KEY = credentials('flask-secret-key')
        SONAR_TOKEN = credentials('sonar-token')
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:your-org/flask-app.git'
            }
        }

        stage('Create .env for Docker Compose') {
            steps {
                sh '''
                echo "MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}" > .env
                echo "MYSQL_DATABASE=${MYSQL_DATABASE}" >> .env
                echo "MYSQL_USER=${MYSQL_USER}" >> .env
                echo "MYSQL_PASSWORD=${MYSQL_PASSWORD}" >> .env
                echo "FLASK_SECRET_KEY=${FLASK_SECRET_KEY}" >> .env
                '''
            }
        }

        stage('Deploy to EC2-2') {
            steps {
                sshagent(['ec2-2-ssh-key']) {
                    sh '''
                    scp -r * ${DEPLOY_USER}@${DEPLOY_HOST}:/home/ubuntu/flask-app/
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'cd /home/ubuntu/flask-app && docker-compose down && docker-compose up -d --build'
                    '''
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                sshagent(['ec2-2-ssh-key']) {
                    sh '''
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                      sonar-scanner \
                      -Dsonar.projectKey=flask-app \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://localhost:9000 \
                      -Dsonar.token=${SONAR_TOKEN}
                    '
                    '''
                }
            }
        }
    }
}
