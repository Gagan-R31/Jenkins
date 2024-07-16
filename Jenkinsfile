pipeline {
    agent any
    triggers {
        githubPush()
    }

    stages {
        stage('Run Script') {
            steps {
                script {
                sh 'python3 test_script.py'
                }
            }
        }
    }
}
