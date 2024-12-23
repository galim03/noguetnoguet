pipeline {
    agent any
    environment {
        PROJECT_ID = 'opensource-437302'       // GCP 프로젝트 ID
        CLUSTER_NAME = 'kube'                  // GKE 클러스터 이름
        LOCATION = 'asia-northeast3'          // 클러스터 위치
        CREDENTIALS_ID = 'gke'                // GCP 인증 정보 (Jenkins에서 설정한 Google 서비스 계정 키 파일)
        DOCKER_IMAGE = "eungga/noguet:${BUILD_ID}"  // Docker 이미지 이름
    }
    stages {
        stage("Checkout code") {
            steps {
                script {
                    // Git 리포지토리에서 코드를 체크아웃합니다.
                    git url: 'https://github.com/galim03/noguetnoguet.git', branch: env.BRANCH_NAME
                }
            }
        }
        stage("Build Docker image") {
            steps {
                script {
                    sh "docker build -t eungga/noguet:${BUILD_ID} ."
                }
            }
        }
        stage('Test Docker Image') {
            steps {
                script {
                    try {
                        sh 'docker run -d -p 3030:3030 --name noguet_container eungga/noguet:${BUILD_ID}'
                        sh 'sleep 5 && curl -f http://34.64.227.223:3030 || exit 1'
                        echo "Container is running correctly."
                    } catch (Exception e) {
                        echo "Test failed. Image will not be pushed."
                        error "Stopping pipeline due to test failure."
                    }
                }
            }
        }
        stage("Push Docker image") {
            when {
                branch 'main'  // main 브랜치일 때만 실행
            }
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-credentials', url: 'https://index.docker.io/v1/']) {
                        sh "docker push eungga/noguet:${BUILD_ID}"
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            when {
                branch 'main'  // main 브랜치일 때만 실행
            }
            steps {
                script {
                    sh "sed -i 's/eungga\\/noguet:latest/eungga\\/noguet:${BUILD_ID}/g' deployment.yaml"
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
    }
    post {
        always {
            script {
                sh 'docker stop noguet_container || true'
                sh 'docker rm noguet_container || true'
            }
            echo 'Pipeline completed.'
        }
        failure {
            script {
                echo "Build failed. Deleting the Docker image."
                sh 'docker rmi $DOCKER_IMAGE || true'
            }
        }
        success {
            echo 'Pipeline succeeded!'
        }
    }
}

