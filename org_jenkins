pipeline {
    agent any

    environment {
        PROD_SERVER = "ec2-user@3.79.3.54"
        APP_NAME = "myapp"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Deploy to Production') {
            steps {
                sshagent(['prod-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $PROD_SERVER '
                        docker stop $APP_NAME || true &&
                        docker rm $APP_NAME || true &&
                        docker run -d -p 80:80 --name $APP_NAME myapp:latest
                    '
                    """
                }
            }
        }
    }
}
