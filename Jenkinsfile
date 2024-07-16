pipeline {
    agent any
    triggers {
        githubPush() // Trigger the pipeline on GitHub push event
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
                    sh 'rm -rf *' // Clean up the workspace
                }
            }
        }
        stage('Checkout') {
            steps {
                script {
                    checkout scm // Check out the source code
                }
            }
        }
        stage('Run Script') {
            steps {
                script {
                    sh 'python3 test_script.py' // Run the Python script
                }
            }
        }
    }
    post {
        always {
            script {
                closeExistingPullRequests() // Close existing PRs always
            }
        }
        success {
            script {
                createNewPullRequest() // Create a new PR on success
            }
        }
        failure {
            script {
                updateGitHubStatus("failure", "Pipeline failed") // Update GitHub status on failure
            }
        }
    }
}

def closeExistingPullRequests() {
    try {
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
    } catch (Exception e) {
        echo "Error in closeExistingPullRequests: ${e.message}"
    }
}

def createNewPullRequest() {
    try {
        def currentBranch = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
        echo "Current branch: ${currentBranch}"
        
        // Ensure the branch exists on the remote
        sh "git push origin ${currentBranch}"
        
        def currentDate = new Date().format("yyyyMMdd-HHmmss")
        def prTitle = "Merge ${currentBranch} into main - ${currentDate}"
        
        def response = sh(script: """
            curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
            -H "Accept: application/vnd.github.v3+json" \
            https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls \
            -d '{
                "title": "${prTitle}",
                "head": "${REPO_OWNER}:${currentBranch}",
                "base": "main"
            }'
        """, returnStdout: true).trim()
        
        echo "API Response: ${response}"
        
        def pullRequest = parseJSON(response)
        if (pullRequest.number) {
            PR_NUMBER = pullRequest.number
            echo "Created new pull request #${PR_NUMBER}"
            updateGitHubStatus("success", "Pipeline succeeded")
        } else {
            echo "Failed to create pull request. Response: ${response}"
            updateGitHubStatus("failure", "Failed to create pull request")
        }
    } catch (Exception e) {
        echo "Error in createNewPullRequest: ${e.message}"
        updateGitHubStatus("failure", "Error creating pull request")
    }
}

def updateGitHubStatus(state, description) {
    try {
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
    } catch (Exception e) {
        echo "Error in updateGitHubStatus: ${e.message}"
    }
}

@NonCPS
def parseJSON(String json) {
    return new groovy.json.JsonSlurperClassic().parseText(json)
}
