pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                        // Checkout the 'TEST' branch
                        sh 'git clone https://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@github.com/Gagan-R31/Jenkins.git -b TEST'
                    }
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
                expression { env.BRANCH_NAME == 'TEST' }
            }
            steps {
                script {
                    def response = sh(script: '''
                        curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
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

