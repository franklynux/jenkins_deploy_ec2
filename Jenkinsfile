pipeline {
    agent any

    environment {
        EC2_IP = '54.91.89.21'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'franklynux/e-commerce-web:${BUILD_NUMBER}'
        EC2_INSTANCE_KEY = credentials('ec2-instance-key')
    }

    stages {
        stage ('fetch code') {
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

        stage('Create Dockerfile') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: '''
                        FROM ubuntu:20.04
                        RUN apt-get update && apt-get install -y wget unzip apache2
                        COPY websetup.sh /websetup.sh
                        RUN chmod +x /websetup.sh
                        RUN /websetup.sh
                        EXPOSE 80
                        CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
                    '''
                }
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
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}"
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
    }
}