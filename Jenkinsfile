pipeline {
    // Use the official Jenkins image as the default agent
    agent {
        docker {
            image 'jenkins/jenkins:lts'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root' // Run as root to avoid Docker socket permission issues
        }
    }

    environment {
        DOCKER_IMAGE = 'nextjs-app'          // Local image name
        AZURE_VM_IP = '20.198.21.95'        // Replace with your Azure VM's public IP
        SSH_CREDENTIALS_ID = 'azure-vm-ssh' // Jenkins credential ID for SSH
        GIT_CREDENTIALS_ID = 'github-credentials' // GitHub credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                // Ensure the workspace is clean and clone the repository
                cleanWs()
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Rafael00X/jenkins-test.git',
                        credentialsId: env.GIT_CREDENTIALS_ID
                    ]]
                ])
                // Verify the Git repository is cloned
                sh 'git status'
            }
        }

        stage('Build Next.js') {
            // Use a Node.js container for this stage
            agent {
                docker {
                    image 'node:20' // Official Node.js 20 image
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm install
                    npm run build
                '''
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
                        ssh -o StrictHostKeyChecking=no user@${AZURE_VM_IP} << 'EOF'
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

    post {
        always {
            // Clean up workspace to avoid clutter
            cleanWs()
        }
        failure {
            echo 'Build or deployment failed!'
        }
        success {
            echo 'Build and deployment completed successfully!'
        }
    }
}
