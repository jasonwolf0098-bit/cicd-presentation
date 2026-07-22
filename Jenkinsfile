pipeline {
    agent any
    
    environment {
        // نام کاربری داکرهاب شما اعمال شد
        DOCKER_IMAGE = 'jasonwolf0098/cicd-presentation'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials' // آیدی تنظیم شده در جنکینز برای داکرهاب
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
                    
                    // اعمال فایل‌ها روی کلاستر
                    sh 'KUBECONFIG=/var/jenkins_home/kubeconfig kubectl apply -f deployment.yaml'
                    sh 'KUBECONFIG=/var/jenkins_home/kubeconfig kubectl apply -f service.yaml'
                }
            }
        }
    }
}
