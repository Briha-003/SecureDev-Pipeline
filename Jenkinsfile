pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Add other stages like build, test, deploy here if needed
    }

    post {
        always {
            // Run inside a node context to provide workspace for junit
            node {
                // Run junit only if test results exist
                script {
                    def testResults = findFiles(glob: '**/target/surefire-reports/*.xml')
                    if (testResults.length > 0) {
                        junit '**/target/surefire-reports/*.xml'
                    } else {
                        echo "No test results found to archive."
                    }
                }
            }
        }
        failure {
            node {
                // Send mail only if SMTP is configured properly
                // Update the 'to' and 'from' fields as per your environment
                mail to: 'team@example.com',
                     subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Check Jenkins console output at ${env.BUILD_URL}"
            }
        }
    }
}
