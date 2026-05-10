pipeline {
    agent any

    environment {
        EC2_HOST = "ec2-user@13.233.127.174"
        SSH_KEY = "/var/lib/jenkins/.ssh/aws_key.pem"
        APP_DIR = "/home/ec2-user/myapp"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SINGHAYUSH24/Sample.git'
            }
        }

        stage('Transfer Source Code') {
            steps {
                sh '''
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_HOST "mkdir -p $APP_DIR"

                rsync -avz --delete \
                  --exclude='.git' \
                  --exclude='myapp.tar' \
                  -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=no" \
                  ./ $EC2_HOST:$APP_DIR/
                '''
            }
        }

        stage('Build and Deploy on EC2') {
            steps {
                sh '''
ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_HOST << EOF
cd $APP_DIR

docker build --no-cache -t myapp:latest .

docker stop myapp || true
docker rm myapp || true

docker run -d \
  --name myapp \
  -p 80:80 \
  --restart unless-stopped \
  myapp:latest

docker image prune -f
EOF
                '''
            }
        }
    }
}