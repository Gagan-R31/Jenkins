pipeline {
    agent any

    stages {
        stage('Cleanup') {
            steps {
                script {
                    // Remove the existing directory if it exists
                    sh 'rm -rf *'
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    // Checkout the 'TEST' branch
                    checkout([$class: 'GitSCM', branches: [[name: '*/TEST']],
                              userRemoteConfigs: [[url: "https://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@github.com/Gagan-R31/Jenkins.git"]]])
                }
            }
        }

        stage('Run Script') {
            steps {
                script {
                    // Ensure the script runs from the correct path
                    sh 'python3 test_script.py'
                }
            }
        }

        stage('Create Pull Request') {
            when {
                branch 'TEST'
            }
            steps {
                script {
                    def response = sh(script: '''
                        curl -X POST -H "Authorization: token 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" \
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
