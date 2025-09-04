pipeline {
    agent {
        label 'dual'
    }

    environment {
        IMAGE_TAG = "buid-${BUILD_NUMBER}"
        DOCKER_NETWORK = 'my-app-network'
        AGENT_PUBLIC_IP = '98.82.1.41'    
    }

    stages {
        stage('Checkout') {
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
                    steps {
                        dir('netflix') {
                            // For static websites, the build artifact is the entire directory
                            sh 'echo "Building Netflix static content"'
                            // Archive the entire built content as artifact
                            archiveArtifacts artifacts: '**/*', fingerprint: true
                        }
                    }
                }
                stage('Build Starbucks') {
                    steps {
                        dir('starbucks') {
                            sh 'echo "Building Starbucks static content"'
                            archiveArtifacts artifacts: '**/*', fingerprint: true
                        }
                    }
                }
                stage('Build NodeJS') {
                    steps {
                        dir('nodejs') {
                            // For Node.js, build the application first
                            sh 'npm install'
                            sh 'npm run build'
                            // Archive the dist folder and package.json as artifacts
                            archiveArtifacts artifacts: 'dist/**/*, package.json, package-lock.json', fingerprint: true
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Netflix Image') {
                    steps {
                        dir('netflix') {
                            script {
                                sh """
                                    docker build -t netflix:${IMAGE_TAG} .
                                    docker tag netflix:${IMAGE_TAG} netflix:latest
                                """
                            }
                        }
                    }
                }
                stage('Starbucks Image') {
                    steps {
                        dir('starbucks') {
                            script {
                                sh """
                                    docker build -t starbucks:${IMAGE_TAG} .
                                    docker tag starbucks:${IMAGE_TAG} starbucks:latest
                                """
                            }
                        }
                    }
                }
                stage('NodeJS Image') {
                    steps {
                        dir('nodejs') {
                            script {
                                sh """
                                    docker build -t nodejs:${IMAGE_TAG} .
                                    docker tag nodejs:${IMAGE_TAG} nodejs:latest
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Archive Docker Images') {
            steps {
                script {
                    // Save Docker images as tar files (alternative artifact format)
                    sh """
                        docker save -o netflix-${IMAGE_TAG}.tar netflix:${IMAGE_TAG}
                        docker save -o starbucks-${IMAGE_TAG}.tar starbucks:${IMAGE_TAG}
                        docker save -o nodejs-${IMAGE_TAG}.tar nodejs:${IMAGE_TAG}
                    """
                    // Archive the Docker image tar files
                    archiveArtifacts artifacts: '*.tar', fingerprint: true
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    sh """
                        docker network inspect ${DOCKER_NETWORK} >/dev/null 2>&1 || docker network create ${DOCKER_NETWORK}
                        docker stop netflix starbucks nodejs || true
                        docker rm netflix starbucks nodejs || true

                        docker run -d --name netflix --network ${DOCKER_NETWORK} -p 8081:80 netflix:latest
                        docker run -d --name starbucks --network ${DOCKER_NETWORK} -p 8082:80 starbucks:latest
                        docker run -d --name nodejs --network ${DOCKER_NETWORK} -p 3000:3000 nodejs:latest
                    """
                }
            }
        }
    }

    post {
        always {
            // // Generate build summary
            // sh """
            //     echo "Build Number: ${BUILD_NUMBER}" > build-info.txt
            //     echo "Image Tag: ${IMAGE_TAG}" >> build-info.txt
            //     echo "Build Date: \$(date)" >> build-info.txt
            //     echo "Git Commit: \$(git rev-parse HEAD)" >> build-info.txt
            // """
            // archiveArtifacts artifacts: 'build-info.txt', fingerprint: true

            echo "Deployment completed. Applications are running:"
            echo "Netflix: http://${AGENT_PUBLIC_IP}:8081"
            echo "StarBucks: http://${AGENT_PUBLIC_IP}:8082"
            echo "Node JS: http://${AGENT_PUBLIC_IP}:3000"
            
        //     // Clean up temporary files
        //     sh 'rm -f *.tar build-info.txt || true'
        // }
        // success {
            // Additional success actions
            echo "All artifacts created and deployed successfully!"
        }
    }
}