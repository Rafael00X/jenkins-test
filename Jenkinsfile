pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nextjs-app'          // Local image name
        AZURE_VM_IP = '20.198.21.95'        // Replace with your VM's public IP
        SSH_CREDENTIALS_ID = 'azure-vm-ssh' // Jenkins credential ID for SSH
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
                    scp -o StrictHostKeyChecking=no nextjs-app.tar.gz azureuser@20.198.21.95:/home/azureuser/
                    ssh -o StrictHostKeyChecking=no azureuser@20.198.21.95 << EOF
                    docker load -i /home/azureuser/nextjs-app.tar.gz
                    docker stop nextjs-container || true
                    docker rm nextjs-container || true
                    docker run -d -p 80:3000 --name nextjs-container ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                    rm /home/azureuser/nextjs-app.tar.gz
                    EOF
                    """
                }
            }
        }
    }
}
