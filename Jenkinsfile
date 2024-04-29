pipeline {
    agent any
    tools {
        nodejs "Node-21"
    }
    environment {
        EC2_HOST = '15.206.123.104'
        EC2_USER = 'ubuntu'
        PRIVATE_KEY = '/var/lib/jenkins/ridham-ngnix-keypair.pem'
        DOCKER_IMAGE_NAME = 'ridhampatel/jenkins-backend-project'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your Git repository
               script {
                   git branch: 'main', url: 'https://github.com/ridhampatel24/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git', credentialsId: 'github-id' 
                }
            }
        }

        stage('Build React App') {
            steps {
                // Build React app
                sh 'cd public && npm install && npm run build && cd build && pwd'
            }
        }
        
        stage('Transfer Frotend to EC2') {
            steps {
                script {
                    sh "rsync -avrx -e 'ssh -i ${PRIVATE_KEY} -o StrictHostKeyChecking=no' --delete /var/lib/jenkins/workspace/ridham-ec2.prod/public/build/ ${EC2_USER}@${EC2_HOST}:/var/www/html/chatapp"                  
                }
            }
        }
        
        stage('Build and push Docker Image') {
            steps {
                script {
                     // Build Docker image for Node Backend
                    sh 'whoami'
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:nodebackend", " ./server")
                    docker.withRegistry( '', 'docker-ridham' ) {  
                        dockerImage.push("nodebackend")
                    }
                }
            }
        }        
        
        stage('Run Docker Image on AWS EC2') {
            steps {
                script {
                    // This command will delete any contianer running on 5000 so this new docker container run easily.
                    def commands = """
                        cd 
                        docker rmi -f ridhampatel/jenkins-backend-project || true
                        docker rm -f \$(docker ps -q --filter "publish=5000/tcp")
                        docker run -d -p 5000:5000 ridhampatel/jenkins-backend-project:nodebackend
                        sudo systemctl restart nginx
                    """
                    // SSH into EC2 instance and pull Docker image
                    sshagent(['ec2-ssh']) {
                    sh "ssh -o StrictHostKeyChecking=no -i ${PRIVATE_KEY} ${EC2_USER}@${EC2_HOST} '${commands}'"
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded. Node.js app deployed to AWS EC2.'
        }
        failure {
            echo 'Pipeline failed. Check the logs for errors.'
        }
    }
}
