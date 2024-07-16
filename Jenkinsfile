pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Gagan-R31/Jenkins.git'
            }
        }

        stage('Run Script') {
            steps {
                sh 'python3 test_script.py'
            }
        }
    }
}
