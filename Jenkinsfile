pipeline {
    agent any

    environment {
        DOCKER_HOST_IP = "13.51.40.89"
        DOCKER_USER = "ubuntu"
        REMOTE_APP_DIR = "blog-app"
        APP_NAME = "blog-frontend"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rohitbirje45/frontend.git'
            }
        }

        stage('Build & Deploy on Remote EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                    # 1) Copy all source files to EC2
                    scp -i \$KEY -o StrictHostKeyChecking=no -r * ${DOCKER_USER}@${DOCKER_HOST_IP}:~/${REMOTE_APP_DIR}/

                    # 2) SSH into EC2 and run “build” + “serve” via PM2
                    ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                        cd ~/${REMOTE_APP_DIR} &&

                        # (a) Install Node + build for production
                        npm install &&
                        npm run build &&

                        # (b) Install “serve” (if not already installed)
                        command -v serve >/dev/null 2>&1 || npm install -g serve &&

                        # (c) Stop any existing PM2 instance (to free port 3000)
                        pm2 delete ${APP_NAME} || true &&

                        # (d) Start “serve” to serve the build/ folder on port 3000
                        pm2 start serve --name "${APP_NAME}" -- -s build -l 3000 &&

                        # (e) Save PM2 process list + enable auto‐startup on reboot
                        pm2 save &&
                        pm2 startup systemd -u ubuntu --hp /home/ubuntu | tail -n 1 | bash
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
