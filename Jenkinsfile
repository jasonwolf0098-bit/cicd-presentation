pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'jasonwolf0098/cicd-presentation'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${TAG}"
                    }
                }
            }
        }
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    // آپدیت کردن تگ ایمیج در فایل deployment
                    sh "sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${TAG}|g' deployment.yaml"
                    
                    // استفاده از فایل کانفیگ امن شده در جنکینز برای اتصال
                    withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml'
                        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f service.yaml'
                    }
                }
            }
        }
    }
}
