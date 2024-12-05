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
            when {
                branch 'main'
            }
            step {
                sh "sed -i 's/hello:latest/eungga/noguet:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', 
                      projectId: env.PROJECT_ID, 
                      clusterName: env.CLUSTER_NAME,
                      location: env.LOCATION, 
                      manifestPattern: 'deployment.yaml', 
                      credentialsId: env.CREDENTIALS_ID,
                      verifyDeployments: true])
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
            echo 'noguet deploy test!!'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
