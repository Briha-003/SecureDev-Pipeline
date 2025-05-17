pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS_16', type: 'NodeJS'
        PATH = "${env.NODEJS_HOME}/bin:${env.PATH}"
        DOCKER_IMAGE = "yourdockerhubusername/securedev-bugtracker"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/yourusername/yourrepo.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'npm test'  // Assuming you run tests via "npm test" using Mocha or similar
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest || true'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--failOnCVSS 7', odcInstallation: 'DependencyCheck'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push Docker Image') {
            when {
                expression { return env.BRANCH_NAME == 'main' }
            }
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker-compose down'
                sh 'docker-compose up -d --build'
            }
        }
    }

    post {
        always {
            junit 'reports/**/*.xml'  // Adjust according to your test reports location
            cleanWs()
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Build failed in Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Please check the Jenkins job for details."
        }
    }
}
