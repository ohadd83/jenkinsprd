pipeline {
    agent any

    environment {
        IMAGE_NAME = "ohadd306/simple-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EC2_HOST = "18.199.237.108"
    }

    stages {

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-prod-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${EC2_HOST}'
                        
                        docker pull ${IMAGE_NAME}:${IMAGE_TAG}

                        docker stop app || true
                        docker rm app || true

                        docker run -d -p 80:3000 --name app ${IMAGE_NAME}:${IMAGE_TAG}

                    '
                    """
                }
            }
        }
    }
}
