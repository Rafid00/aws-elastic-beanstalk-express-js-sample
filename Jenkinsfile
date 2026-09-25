pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm ci'
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh 'npm test'
            }
        }

        stage('Dependency Security Scan') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    set +e
                    npm audit --audit-level=high --json > npm-audit.json
                    status=$?
                    set -e
                    cat npm-audit.json
                    exit $status
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t isec6000-node-app:${BUILD_NUMBER} -t isec6000-node-app:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        printf '%s' "$DOCKER_TOKEN" | docker login --username "$DOCKER_USER" --password-stdin

                        docker tag "isec6000-node-app:${BUILD_NUMBER}" "$DOCKER_USER/isec6000-assignment-2-node-app:${BUILD_NUMBER}"
                        docker tag "isec6000-node-app:latest" "$DOCKER_USER/isec6000-assignment-2-node-app:latest"

                        docker push "$DOCKER_USER/isec6000-assignment-2-node-app:${BUILD_NUMBER}"
                        docker push "$DOCKER_USER/isec6000-assignment-2-node-app:latest"

                        docker logout
                    '''
                }
            }
        }
    }
}
