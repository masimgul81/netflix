pipeline {
    agent none

    environment {
        IMAGE_TAG = "buid-${BUILD_NUMBER}"
        DOCKER_NETWORK = 'my-app-network'
        AGENT_PUBLIC_IP = '20.120.98.12'    
    }

    stages {
        stage('Checkout') {
            agent { label 'dual' }
            steps {
                dir('netflix') {
                    git branch: 'main', url: 'https://github.com/masimgul81/netflix.git'
                }
                dir('starbucks') {
                    git branch: 'main', url: 'https://github.com/masimgul81/starbucks.git'
                }
                dir('nodejs') {
                    git branch: 'main', url: 'https://github.com/masimgul81/nodejs.git'
                }
            } 
        }

        stage('Build Projects') {
            parallel {
                stage('Build Netflix') {
                    agent { label 'dual' }
                    steps {
                        dir('netflix') {
                            sh 'echo "Building Netflix static content"'
                            archiveArtifacts artifacts: '**/*', fingerprint: true
                        }
                    }
                }
                stage('Build Starbucks') {
                    agent { label 'dual' }
                    steps {
                        dir('starbucks') {
                            sh 'echo "Building Starbucks static content"'
                            script {
                                def files = findFiles(glob: '**/*')
                                if (files) {
                                    archiveArtifacts artifacts: '**/*', fingerprint: true
                                } else {
                                    echo "No files found to archive in Starbucks"
                                }
                            }
                        }
                    }
                }
                stage('Build NodeJS') {
                    agent { label 'dual' }
                    steps {
                        dir('nodejs') {
                            script {
                                if (fileExists('package.json')) {
                                    docker.image('node:18-alpine').inside {
                                        sh 'npm install'
                                        sh 'npm run build'
                                    }
                                    archiveArtifacts artifacts: 'dist/**/*, package.json, package-lock.json', fingerprint: true
                                } else {
                                    error "package.json not found! Files: ${sh(script: 'ls -la', returnStdout: true)}"
                                }
                            }
                        }
                    }
                }
            }
        }

        // ... keep the rest of your stages the same ...
    }

    post {
        always {
            // REMOVE THE 'steps' BLOCK - put commands directly here
            sh """
                echo "Build Number: ${BUILD_NUMBER}" > build-info.txt
                echo "Image Tag: ${IMAGE_TAG}" >> build-info.txt
                echo "Build Date: \$(date)" >> build-info.txt
                echo "Git Commit: \$(git rev-parse HEAD)" >> build-info.txt
            """
            archiveArtifacts artifacts: 'build-info.txt', fingerprint: true

            echo "Deployment completed. Applications are running:"
            echo "Netflix: http://${AGENT_PUBLIC_IP}:8081"
            echo "StarBucks: http://${AGENT_PUBLIC_IP}:8082"
            echo "Node JS: http://${AGENT_PUBLIC_IP}:3000"
            
            sh 'rm -f *.tar build-info.txt || true'
        }
    }
}