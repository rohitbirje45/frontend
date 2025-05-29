pipeline {
    agent any

    environment {
        DOCKER_HOST_IP = "13.51.40.89"
        DOCKER_USER = "ubuntu"
        DOCKER_APP_DIR = "blog-app"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rohitbirje45/frontend.git'
            }
        }

        stage('Build Docker Image') {
    steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
            sh """
                ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                    rm -rf ${DOCKER_APP_DIR} && mkdir -p ${DOCKER_APP_DIR}
                '

                scp -i \$KEY -o StrictHostKeyChecking=no -r \
                    src public \
                    Dockerfile package.json package-lock.json yarn.lock\
                    ${DOCKER_USER}@${DOCKER_HOST_IP}:${DOCKER_APP_DIR}/

                ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                    cd ${DOCKER_APP_DIR} &&
                    docker build -t blog-app .
                '
            """
        }
    }
}


        stage('Run Container') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                        ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                            docker rm -f blog-container || true &&
                            docker run -d -p 3000:3000 --name blog-container blog-app
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
