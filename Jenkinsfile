pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'eungga/noguet'  // 도커 이미지 이름
        DOCKER_REGISTRY = 'docker.io'   // 도커 허브 레지스트리
        DOCKER_CREDENTIALS = 'docker-credentials'  // Jenkins에 저장된 Docker 로그인 자격 증명
        TEST_PORT = '3030'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/galim03/noguetnoguet.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    // Docker Hub에 로그인
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS) {
                        // Docker 이미지 푸시
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    // 테스트 서버 실행 (예: 포트 3030)
                    sh "node main.js"
                }
            }
        }
    }
}
