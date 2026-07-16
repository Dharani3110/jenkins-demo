pipeline {

    agent any

    environment {

        IMAGE = "dharani3110/jenkins-demo"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"

    }

    stages {

        stage('Setup Python') {

            steps {

                sh '''
                python3 -m venv venv
                . venv/bin/activate
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
                -t $IMAGE:$BUILD_NUMBER \
                -t $IMAGE:latest .
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
                docker push $IMAGE:$BUILD_NUMBER
                docker push $IMAGE:latest
                '''

            }

        }

        stage('Deploy to Kubernetes') {
 	    steps {
      		sh '''
            	    kubectl set image deployment/flask-app \
                    flask-container=$IMAGE:$BUILD_NUMBER

                    kubectl rollout status deployment/flask-app
                   '''
                  }
               }

            }

        }

    }

    post {

        success {

            echo "Deployment Successful"

        }

        failure {

            echo "Deployment Failed"

        }

        always {

            sh "docker logout || true"

            sh "rm -rf venv || true"

        }

    }

}
