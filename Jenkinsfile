pipeline {
    agent any

    environment {
        DOCKER_HOST_IP  = "13.51.40.89"
        DOCKER_USER     = "ubuntu"
        REMOTE_APP_DIR  = "blog-app"
        APP_NAME        = "blog-frontend"
        SSH_KEY_CRED_ID = "ec2-ssh-key"
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
                    credentialsId: "${SSH_KEY_CRED_ID}",
                    keyFileVariable: 'KEY'
                )]) {
                    sh """
                      # 1) Copy entire workspace to remote host’s ~/blog-app/
                      scp -i \$KEY -o StrictHostKeyChecking=no -r * ${DOCKER_USER}@${DOCKER_HOST_IP}:~/${REMOTE_APP_DIR}/

                      # 2) SSH into remote and run everything there (build + serve)
                      ssh -i \$KEY -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_HOST_IP} << 'EOF'
set -e

cd ~/${REMOTE_APP_DIR}

# --- (a) Clean up any previous build or PM2 process --- 
pm2 delete ${APP_NAME} 2>/dev/null || true
rm -rf node_modules build

# --- (b) Install dependencies on remote ---
npm install

# --- (c) Build with higher heap size to avoid OOM ---
export NODE_OPTIONS="--max_old_space_size=1024"
npm run build

# --- (d) Install serve locally (in node_modules) to avoid sudo issues ---
npm install serve --no-save

# --- (e) Start the static build under PM2 on remote port 3000 ---
pm2 start npx --name "${APP_NAME}" -- serve -s build -l 3000

# --- (f) Persist PM2 state & enable auto-start on reboot ---
pm2 save
pm2 startup systemd -u ubuntu --hp /home/ubuntu | tail -n 1 | bash
EOF
                    """
                }
            }
        }

        stage('Selenium Tests') {
            steps {
                sh """
                  echo "Running Selenium tests against http://${DOCKER_HOST_IP}:3000"
                  # TODO: replace with: pytest or npm run test or your Selenium command
                """
            }
        }
    }
}
