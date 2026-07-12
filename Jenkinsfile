pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Application') {
            steps {
                sh 'python3 app.py'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 test.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-demo:v1 .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker rm -f demo 2>/dev/null || true
                    docker run -d --name demo jenkins-demo:v1
                '''
            }
        }

    }
}
