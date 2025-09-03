pipeline {
    agent any  // Runs on the Jenkins agent (your VM)

    environment {
        DOCKER_IMAGE = 'nextjs-app'  // Name your image
        CONTAINER_NAME = 'nextjs-container'  // Name your container
        PORT = '3000'  // Exposed port
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Build Next.js') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Deploy to Docker') {
            steps {
                // Stop and remove existing container if running
                sh """
                if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME}
                fi
                if [ \$(docker ps -a -q -f name=${CONTAINER_NAME}) ]; then
                    docker rm ${CONTAINER_NAME}
                fi
                """
                // Run new container
                sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:3000 ${DOCKER_IMAGE}:latest"
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
