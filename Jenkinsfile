pipeline {
    agent any
    environment {
        PROJECT_ID = 'opensource-437302'
        CLUSTER_ID = 'kube'
        LOCATION = 'asia-northeast3'
        CREDENTIAL_ID = 'gke'
        DOCKER_IMAGE = 'eungga/noguet:latest'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/galim03/noguetnoguet.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Docker 컨테이너에서 main.js 실행
                    sh 'docker run -d -p 3030:3030 --name noguet_container $DOCKER_IMAGE'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-credentials', url: 'https://index.docker.io/v1/']) {
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo 'Deploying to GKE'
                script {
                    // GCP 인증
                    withCredentials([file(credentialsId: CREDENTIAL_ID, variable: 'GCP_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GCP_CREDENTIALS'
                        sh 'gcloud container clusters get-credentials $CLUSTER_ID --zone $LOCATION --project $PROJECT_ID'
                    }
                    // Kubernetes 배포
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            // 컨테이너가 실행 중이면 종료
            script {
                sh 'docker stop noguet_container || true'
                sh 'docker rm noguet_container || true'
            }
            echo 'Pipeline completed.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

