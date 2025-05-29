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

        stage('Deploy & Run with PM2 on Remote EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'KEY')]) {
                    sh """
                    # Copy all source files to EC2
                    scp -i \$KEY -o StrictHostKeyChecking=no -r * ${DOCKER_USER}@${DOCKER_HOST_IP}:~/${REMOTE_APP_DIR}/

                    # SSH into EC2 and run commands
                    ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} '
                        # Go to project directory
                        cd ~/${REMOTE_APP_DIR} &&
                        
                        # Install Node modules
                        npm install &&


                        # Start the app with pm2
                        pm2 start npm --name "${APP_NAME}" -- start -- --host 0.0.0.0 &&

                        # Save the process list
                        pm2 save &&

                        # Enable pm2 startup on reboot
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
