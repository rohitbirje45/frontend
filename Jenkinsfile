pipeline {
    agent any

    environment {
        DOCKER_HOST_IP  = "13.51.40.89"
        DOCKER_USER     = "ubuntu"
        REMOTE_APP_DIR  = "blog-app"
        APP_NAME        = "blog-frontend"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rohitbirje45/frontend.git'
            }
        }

        stage('Copy & Build on Remote EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key',
                    keyFileVariable: 'KEY'
                )]) {
                    sh '''
                      # 1) Copy entire workspace to the remote host
                      scp -i $KEY -o StrictHostKeyChecking=no -r * ${DOCKER_USER}@${DOCKER_HOST_IP}:~/${REMOTE_APP_DIR}/

                      # 2) SSH into remote host and do everything (install, build, serve)
                      ssh -i $KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} << 'EOF'
                        set -e

                        cd ~/${REMOTE_APP_DIR}

                        # Clean up any old builds/processes
                        pm2 delete ${APP_NAME} || true
                        rm -rf build
                        rm -rf node_modules

                        # (a) Install dependencies
                        npm install

                        # (b) Build with increased heap size to avoid OOM on small EC2
                        export NODE_OPTIONS="--max_old_space_size=1024"
                        npm run build

                        # (c) Install `serve` locally (in node_modules) to avoid sudo issues
                        npm install serve --no-save

                        # (d) Start the static build under PM2 on port 3000
                        pm2 start npx --name "${APP_NAME}" -- serve -s build -l 3000

                        # (e) Save PM2 list & enable auto startup on reboot
                        pm2 save
                        pm2 startup systemd -u ubuntu --hp /home/ubuntu | tail -n 1 | bash
                      EOF
                    '''
                }
            }
        }

        stage('Selenium Tests') {
            steps {
                sh '''
                  echo "Running Selenium tests against http://${DOCKER_HOST_IP}:3000"
                  # TODO: actually invoke your Selenium suite here, e.g.:
                  #   pytest tests/selenium --base-url=http://${DOCKER_HOST_IP}:3000
                '''
            }
        }
    }
}
