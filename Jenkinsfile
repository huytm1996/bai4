pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_APP = 'huytm1996/app4'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                // Jenkins tự clone đúng branch đang chạy
                checkout scm
                sh 'echo "Current branch: ${BRANCH_NAME}"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image for branch: ${BRANCH_NAME}"
                docker build -t $IMAGE_APP:${BRANCH_NAME} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push $IMAGE_APP:${BRANCH_NAME}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def ns = "${BRANCH_NAME}"
                    sh """
                     kubectl delete pod -l app=app4 -n ${ns}
                    kubectl create namespace ${ns} --dry-run=client -o yaml | kubectl apply -f -
                    kubectl apply -f deployment-${ns}.yaml -n ${ns}
                    kubectl set image deployment/deployment-${ns} app4=$IMAGE_APP:${BRANCH_NAME} -n ${ns} || true
                    kubectl rollout status deployment/deployment-${ns} -n ${ns}
                    """
                }
            }
        }
    }
     triggers {
       
        githubPush()
    }
}
