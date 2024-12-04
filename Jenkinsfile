pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'eungga/noguet'  // 도커 이미지 이름
        DOCKER_REGISTRY = 'docker.io'   // 도커 허브 레지스트리
        DOCKER_CREDENTIALS = 'docker-credentials'  // Jenkins에 저장된 Docker 로그인 자격 증명
        TEST_PORT = '3030'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Git 리포지토리에서 코드 체크아웃
                git branch: 'main', url: 'https://github.com/galim03/noguetnoguet.git'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // 도커 이미지 빌드
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    // 도커 허브 로그인
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS) {
                        // 로그인 후 작업이 진행됨
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    // 도커 이미지를 도커 허브에 푸시
                    sh "docker push ${DOCKER_IMAGE}:pushtest"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // 도커 이미지를 기반으로 컨테이너 실행
                    sh "docker run -d -p ${TEST_PORT}:${TEST_PORT} --name test-container ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Test Docker Container') {
            steps {
                script {
                    // 서버가 정상적으로 동작하는지 확인하기 위해 HTTP 요청을 보냄
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${TEST_PORT}", returnStdout: true).trim()

                    // HTTP 상태 코드가 200번대인 경우 정상적으로 동작한 것으로 판단
                    if (response != '200') {
                        error "테스트 실패: 서버가 정상적으로 동작하지 않습니다. HTTP 응답 코드: ${response}"
                    } else {
                        echo "서버가 정상적으로 동작하고 있습니다. HTTP 응답 코드: ${response}"
                    }
                }
            }
        }

        stage('Cleanup Docker Container') {
            steps {
                script {
                    // 테스트 후 실행된 컨테이너 제거
                    sh "docker stop test-container"
                    sh "docker rm test-container"
                }
            }
        }
    }

    post {
        success {
            echo '도커 이미지가 도커 허브에 성공적으로 푸시되었고, 서버가 정상적으로 동작합니다.'
        }

        failure {
            echo '도커 이미지 푸시 또는 서버 동작 테스트 중 오류가 발생했습니다.'
        }
    }
}
