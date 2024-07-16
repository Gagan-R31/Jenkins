pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the 'TEST' branch
                    checkout([$class: 'GitSCM', branches: [[name: '*/TEST']],
                              userRemoteConfigs: [[url: 'https://github.com/Gagan-R31/Jenkins']]])
                }
            }
        }

        stage('Run Script') {
            steps {
                sh 'python3 test_script.py'
            }
        }

        stage('Create Pull Request') {
            when {
                branch 'TEST'
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-token', variable: 'Jenkins-creds')]) {
                        def response = sh(script: '''
                            curl -X POST -H "Authorization: token ${Jenkins-creds}" \
                            -d \'{
                                "title": "Merge TEST into main",
                                "head": "TEST",
                                "base": "main"
                            }\' \
                            https://api.github.com/repos/Gagan-R31/Jenkins/pulls
                        ''', returnStdout: true).trim()
                        
                        echo "Pull Request Response: ${response}"
                    }
                }
            }
        }
    }
}
