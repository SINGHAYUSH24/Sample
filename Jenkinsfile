pipeline {
    agent any

    environment {
        EC2_HOST = "ec2-user@3.111.186.32"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SINGHAYUSH24/Sample.git'
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
                scp -o StrictHostKeyChecking=no myapp.tar ec2-user@3.111.186.32:/home/ec2-user/
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                ssh ec2-user@3.111.186.32 << 'EOF'
                docker load < myapp.tar
                docker stop myapp || true
                docker rm myapp || true
                docker run -d -p 80:3000 --name myapp myapp:latest
                EOF
                '''
            }
        }
    }
}