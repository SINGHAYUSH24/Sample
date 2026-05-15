pipeline {
    agent any

    environment {
        IMAGE_NAME = "myapp"
        IMAGE_TAG  = "latest"

        EC2_HOST = "ec2-user@15.206.67.42"
        SSH_KEY  = "/var/lib/jenkins/.ssh/aws_key.pem"
        REMOTE_DIR = "/home/ec2-user/app"
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
                sh '''
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Export Docker Image') {
            steps {
                sh '''
                    docker save $IMAGE_NAME:$IMAGE_TAG | gzip > ${IMAGE_NAME}.tar.gz
                '''
            }
        }

        stage('Transfer Image to EC2') {
            steps {
                sh '''
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_HOST \
                        "mkdir -p $REMOTE_DIR"

                    scp -i $SSH_KEY -o StrictHostKeyChecking=no \
                        ${IMAGE_NAME}.tar.gz \
                        $EC2_HOST:$REMOTE_DIR/
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_HOST << EOF
                    set -e

                    cd $REMOTE_DIR
                    gunzip -c ${IMAGE_NAME}.tar.gz | docker load

                    docker stop myapp || true
                    docker rm myapp || true

                    docker run -d \
                      --name myapp \
                      -p 80:80 \
                      --restart unless-stopped \
                      $IMAGE_NAME:$IMAGE_TAG

                    docker image prune -f
                    rm -f ${IMAGE_NAME}.tar.gz
EOF
                '''
            }
        }
    }

    post {
        always {
            sh '''
                rm -f ${IMAGE_NAME}.tar.gz || true
                docker rmi $IMAGE_NAME:$IMAGE_TAG || true
            '''
        }
    }
}