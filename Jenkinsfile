pipeline {
    agent any
    environment {
        IMAGE_NAME = "myapp"
        AWS_ACCOUNT_ID = "123456789012"
        REGION = "us-east-1"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${IMAGE_NAME}"
    }
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/your-org/my-springboot-app.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                sh '''
                aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_URI
                docker build -t $IMAGE_NAME .
                docker tag $IMAGE_NAME:latest $ECR_URI:latest
                docker push $ECR_URI:latest
                '''
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
