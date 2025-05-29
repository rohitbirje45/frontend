pipeline {
    agent any

    environment {
        DOCKER_HOST_IP = "13.51.40.89"
        DOCKER_USER = "ubuntu"
        REMOTE_APP_DIR = "blog-app"
    }

    stages {
        stage('Clone Repository') {
            steps {
                // You can skip cloning locally if you want, or just for archive
                git branch: 'main', url: 'https://github.com/rohitbirje45/frontend.git'
            }
        }

        stage('Deploy & Build on Remote EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    // Copy all source files to remote server
                    sh """
                    scp -i \$KEY -o StrictHostKeyChecking=no -r * ${DOCKER_USER}@${DOCKER_HOST_IP}:~/${REMOTE_APP_DIR}/
                    """

                    // Run npm commands remotely on EC2 inside app dir
                    sh """
                    ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                        cd ~/${REMOTE_APP_DIR} &&
                        npm install &&
                        npm run build
                    '
                    """
                }
            }
        }

        stage('Selenium Tests') {
            steps {
                sh """
                    echo "Running Selenium tests..."
                    # TODO: Add your Selenium test command here
                """
            }
        }
    }
}
