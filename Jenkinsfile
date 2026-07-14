pipeline {

    agent any

    environment {
        IMAGE_NAME = "dharani3110/jenkins-demo"
        CONTAINER_NAME = "jenkins-demo"
        EC2_HOST = "65.0.106.88"
    }

    stages {

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 test.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t $IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST << EOF

                        docker pull $IMAGE_NAME:$BUILD_NUMBER

                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true

                        docker run -d \
                            --name $CONTAINER_NAME \
                            -p 5000:5000 \
                            $IMAGE_NAME:$BUILD_NUMBER

                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }

        failure {
            echo "Pipeline Failed!"
        }

        always {
            sh 'docker logout || true'
            sh 'rm -rf venv || true'
        }
    }
}
