pipeline {
    agent any

    environment {
        EC2_HOST = "ec2-user@13.206.186.119"
        SSH_KEY = "/var/lib/jenkins/.ssh/aws_key.pem"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/SINGHAYUSH24/Sample.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Save Image') {
            steps {
                sh 'docker save myapp:latest > myapp.tar'
            }
        }

        stage('Transfer to EC2') {
            steps {
                sh '''
                scp -i $SSH_KEY \
                -o StrictHostKeyChecking=no \
                myapp.tar \
                $EC2_HOST:/home/ec2-user/
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                ssh -i $SSH_KEY \
                -o StrictHostKeyChecking=no \
                $EC2_HOST << 'EOF'

                docker load < myapp.tar

                docker stop myapp || true
                docker rm myapp || true

                docker run -d \
                -p 80:3000 \
                --name myapp \
                myapp:latest

                EOF
                '''
            }
        }
    }
}