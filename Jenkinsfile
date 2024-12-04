pipeline {
    agent any
    
    tools {
        nodejs '18.15.0'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/galim03/noguetnoguet.git'
            }
        }
        stage('BE-Build') {
            steps {
                dir("./") {
                    sh 'echo "Starting npm install"'
                    sh 'npm install'
                    sh 'echo "npm install complete, running node main.js"'
                    sh 'node main.js'
                }
            }
        }
    }
}
