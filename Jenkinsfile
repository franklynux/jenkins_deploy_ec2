pipeline {
    agent any
    environment {
        EC2_IP = '3.83.179.213'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'franklynux/e-commerce-web:${BUILD_NUMBER}'
    }
    stages {
        stage ('Fetch Code') {
            steps {
                script {
                    echo "Pull source code from Git"
                    git branch: 'main', url: 'https://github.com/franklynux/jenkins_deploy_ec2.git'
                }
            }
        }
       
        stage('Install Python and Dependencies') {
            steps {
                sh '''
                    # Update package list
                    sudo apt-get update
                   
                    # Install Python3 and pip if not already installed
                    sudo apt-get install -y python3 python3-pip
                   
                    # Install BeautifulSoup4
                    pip3 install beautifulsoup4
                '''
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'python3 test_website.py'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                    // Optional: Remove the local image to save space
                    sh "docker rmi ${DOCKER_IMAGE}"
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    def dockerCmd = """
                        docker pull ${DOCKER_IMAGE}
                        docker stop barista-cafe || true
                        docker rm barista-cafe || true
                        docker run -d --name barista-cafe -p 80:80 ${DOCKER_IMAGE}
                    """
                    sshagent(['ec2-server']) {
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} 'sudo apt-get update && sudo apt-get install -y docker.io'"
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '${dockerCmd}'"
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
        success {
            echo 'Pipeline completed successfully!'
            // Optionally, send a success notification (e.g., Slack, Email)
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
            // Optionally, send a failure notification
        }
    }
}