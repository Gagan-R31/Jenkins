pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        GITHUB_TOKEN = "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"
        REPO_OWNER = "Gagan-R31"
        REPO_NAME = "Jenkins"
        PR_NUMBER = ""
    }
    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh 'rm -rf *'
                }
            }
        }
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage('Run Script') {
            steps {
                script {
                    sh 'python3 test_script.py'
                }
            }
        }
    }
    post {
        always {
            script {
                closeExistingPullRequests()
            }
        }
        success {
            script {
                createNewPullRequest()
            }
        }
        failure {
            script {
                updateGitHubStatus("failure", "Pipeline failed")
            }
        }
    }
}

def closeExistingPullRequests() {
    def response = sh(script: """
        curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
        https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls?state=open
    """, returnStdout: true).trim()
    
    def pullRequests = parseJSON(response)
    
    pullRequests.each { pr ->
        if (pr.head.ref == 'TEST' && pr.base.ref == 'main') {
            sh """
                curl -X PATCH -H "Authorization: token ${GITHUB_TOKEN}" \
                -H "Accept: application/vnd.github.v3+json" \
                https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls/${pr.number} \
                -d '{"state":"closed"}'
            """
            echo "Closed pull request #${pr.number}"
        }
    }
}

def createNewPullRequest() {
    def currentDate = new Date().format("yyyyMMdd-HHmmss")
    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
    def prTitle = "Merge ${branchName} into main - ${currentDate}"
    
    def response = sh(script: """
        curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
        -H "Accept: application/vnd.github.v3+json" \
        https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls \
        -d '{
            "title": "${prTitle}",
            "head": "${branchName}",
            "base": "main"
        }'
    """, returnStdout: true).trim()
    
    def pullRequest = parseJSON(response)
    PR_NUMBER = pullRequest.number
    echo "Created new pull request #${PR_NUMBER}"
    
    updateGitHubStatus("success", "Pipeline succeeded")
}

def updateGitHubStatus(state, description) {
    def sha = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
    sh """
        curl -H "Authorization: token ${GITHUB_TOKEN}" \
             -H "Accept: application/vnd.github.v3+json" \
             -X POST \
             https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/statuses/${sha} \
             -d '{
                 "state": "${state}",
                 "target_url": "${BUILD_URL}",
                 "description": "${description}",
                 "context": "continuous-integration/jenkins"
             }'
    """
}

@NonCPS
def parseJSON(String json) {
    return new groovy.json.JsonSlurperClassic().parseText(json)
}
