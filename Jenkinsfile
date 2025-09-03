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
            agent {
                docker {
                    image 'node:18'   // or node:20 if you prefer
                    args '-v $HOME/.npm:/root/.npm' // cache npm between builds
                }
            }
            steps {
                sh 'npm ci'
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
                    scp -o StrictHostKeyChecking=no nextjs-app.tar.gz user@${AZURE_VM_IP}:/home/user/
                    ssh -o StrictHostKeyChecking=no user@${AZURE_VM_IP} << EOF
                    docker load -i /home/user/nextjs-app.tar.gz
                    docker stop nextjs-container || true
                    docker rm nextjs-container || true
                    docker run -d -p 80:3000 --name nextjs-container ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                    rm /home/user/nextjs-app.tar.gz
                    EOF
                    """
                }
            }
        }
    }
}
