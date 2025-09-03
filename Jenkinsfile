pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE       = 'nextjs-app'          // Local image name
        AZURE_VM_IP        = '20.198.21.95'        // VM's public IP
        AZURE_VM_USER      = 'azureuser'           // VM's SSH username
        SSH_CREDENTIALS_ID = 'azure-vm-ssh'        // Jenkins credential ID for SSH
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Rafael00X/jenkins-test.git',
                    branch: 'main',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Next.js') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Save and Transfer Docker Image') {
            steps {
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    sh """
                    docker save ${DOCKER_IMAGE}:${env.BUILD_NUMBER} | gzip > nextjs-app.tar.gz
                    scp -o StrictHostKeyChecking=no nextjs-app.tar.gz ${AZURE_VM_USER}@${AZURE_VM_IP}:/home/${AZURE_VM_USER}/
                    ssh -o StrictHostKeyChecking=no ${AZURE_VM_USER}@${AZURE_VM_IP} "\
                    docker load -i /home/${AZURE_VM_USER}/nextjs-app.tar.gz && \
                    docker stop nextjs-container || true && \
                    docker rm nextjs-container || true && \
                    docker run -d -p 3000:3000 --name nextjs-container ${DOCKER_IMAGE}:${env.BUILD_NUMBER} && \
                    rm /home/${AZURE_VM_USER}/nextjs-app.tar.gz"
                    """
                }
            }
        }
    }
}
