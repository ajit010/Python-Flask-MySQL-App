pipeline {
    agent any
    environment {
        DEPLOY_HOST = '3.137.183.244'
        DEPLOY_USER = 'root'

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
                git branch: 'main', url: 'https://github.com/ajit010/Python-Flask-MySQL-App.git'
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
                withCredentials([usernamePassword(credentialsId: 'ec2-username-password', usernameVariable: 'DEPLOY_USER', passwordVariable: 'DEPLOY_PASSWORD')]) {
                    sh """
                        # Ensure known_hosts is configured to avoid host verification issues
                        mkdir -p ~/.ssh
                        ssh-keyscan -H ${DEPLOY_HOST} >> ~/.ssh/known_hosts

                        # Use sshpass to pass the password for SSH connection
                        sshpass -p ${DEPLOY_PASSWORD} ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                            cd /home/ubuntu/flask-app || git clone https://github.com/ajit010/Python-Flask-MySQL-App.git /home/ubuntu/flask-app
                            cd /home/ubuntu/flask-app
                            git reset --hard
                            git pull origin main
                            docker-compose down
                            docker-compose up -d --build
                        '
                    """
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ec2-username-password', usernameVariable: 'DEPLOY_USER', passwordVariable: 'DEPLOY_PASSWORD')]) {
                    sh '''
                        sshpass -p ${DEPLOY_PASSWORD} ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                            sonar-scanner \
                            -Dsonar.projectKey=flask-app \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://3.144.30.107:9000 \
                            -Dsonar.token=${SONAR_TOKEN}
                        '
                    '''
                }
            }
        }
    }
}
