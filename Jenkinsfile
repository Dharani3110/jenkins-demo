pipeline {

    agent any

    environment {
        IMAGE_NAME = "dharani3110/jenkins-demo"
        CONTAINER_NAME = "jenkins-demo"
        EC2_HOST = "65.0.106.88"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 test.py'
            }
        }

        stage('Build Image') {
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
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                        echo "$PASS" | docker login \
                        -u "$USER" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy') {

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

}
