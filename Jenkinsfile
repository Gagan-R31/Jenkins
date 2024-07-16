pipeline {
    agent any
    triggers{
        githubPush()
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Gagan-R31/Jenkins.git'
            }
        }
        stage('Read Message') {
            steps {
                script {
                    def message = readFile 'message.txt'
                    echo "Message from file: ${message}"
                }
            }
        }
    }
}
