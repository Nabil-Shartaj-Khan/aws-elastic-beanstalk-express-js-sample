pipeline {
    agent none

    stages {

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16'
                    args '-u 1000:1000 -e npm_config_cache=/tmp/.npm'
                }
            }

            steps {
                sh 'npm ci'
            }
        }

        stage('Unit Test') {
            agent {
                docker {
                    image 'node:16'
                    args '-u 1000:1000 -e npm_config_cache=/tmp/.npm'
                }
            }

            steps {
                sh 'npm test'
            }
        }

        stage('Dependency Security Scan') {
            agent {
                docker {
                    image 'node:16'
                    args '-u 1000:1000 -e npm_config_cache=/tmp/.npm'
                }
            }

            steps {
                sh '''
                    npm audit --json > npm-audit.json || true
                    npm audit --audit-level=high
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
                }
            }
        }

        stage('Build Docker Image') {
            agent any

            steps {
                sh '''
                    docker build \
                    -t nabilshartaj/isec6000-node-app:${BUILD_NUMBER} \
                    -t nabilshartaj/isec6000-node-app:latest \
                    .
                '''
            }
        }

        stage('Push Docker Image') {
            agent any

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | \
                        docker login -u "$DOCKERHUB_USER" --password-stdin

                        docker push nabilshartaj/isec6000-node-app:${BUILD_NUMBER}
                        docker push nabilshartaj/isec6000-node-app:latest
                    '''
                }
            }

            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage and logs.'
        }
    }
}
