pipeline {
    agent any

    // Configure a NodeJS installation with this exact name under
    // "Manage Jenkins" > "Global Tool Configuration" > "NodeJS".
    tools {
        nodejs 'NodeJS'
    }

    // This single Jenkinsfile drives both the "main" and "dev" branches.
    // env.BRANCH_NAME is provided automatically when this job is configured
    // as a Multibranch Pipeline (or GitHub Organization) job.
    environment {
        IMAGE_TAG    = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        CONTAINER    = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        HOST_PORT    = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        EXPOSE_PORT  = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Uses the SSH credentials configured on the Jenkins job/
                // multibranch source (Manage Jenkins > Credentials, kind
                // "SSH Username with private key") to pull from GitHub.
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                // Stop/remove the previous container for this branch only
                // right before starting the new one, so downtime is limited
                // to the brief gap between "docker stop" and "docker run".
                sh """
                    docker stop ${CONTAINER} || true
                    docker rm ${CONTAINER} || true
                """
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${CONTAINER} --expose 3000 -p 3000:3000 nodemain:v1.0"
                    } else {
                        sh "docker run -d --name ${CONTAINER} --expose 3001 -p 3001:3000 nodedev:v1.0"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
