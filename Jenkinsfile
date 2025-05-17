pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Add your other build/test stages here
    }

    post {
        always {
            // Just call junit directly without node
            script {
                def testResults = findFiles(glob: '**/target/surefire-reports/*.xml')
                if (testResults.length > 0) {
                    junit '**/target/surefire-reports/*.xml'
                } else {
                    echo "No test results found to archive."
                }
            }
        }

        failure {
            // Call mail directly here
            mail to: 'team@example.com',
                 subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check Jenkins console output at ${env.BUILD_URL}"
        }
    }
}
