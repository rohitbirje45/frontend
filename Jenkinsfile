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
                git branch: 'main', url: 'https://github.com/rohitbirje45/frontend.git'
            }
        }

        stage('Install & Build React App') {
            steps {
                sh """
                    npm install
                    npm run build
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                        ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                            rm -rf ${REMOTE_APP_DIR} && mkdir -p ${REMOTE_APP_DIR}
                        '

                        scp -i \$KEY -o StrictHostKeyChecking=no -r build/* \
                            ${DOCKER_USER}@${DOCKER_HOST_IP}:${REMOTE_APP_DIR}/
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
