pipeline {
    agent none // No global agent - we'll specify per stage

    environment {
        IMAGE_TAG = "buid-${BUILD_NUMBER}"
        DOCKER_NETWORK = 'my-app-network'
        AGENT_PUBLIC_IP = '20.120.98.12'    
    }

    stages {
        stage('Checkout') {
            agent { label 'dual' } // Use any agent with 'dual' label
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
                    agent { label 'dual' } // Jenkins will choose an available agent
                    steps {
                        dir('netflix') {
                            sh 'echo "Building Netflix static content"'
                            archiveArtifacts artifacts: '**/*', fingerprint: true
                        }
                    }
                }
                stage('Build Starbucks') {
                    agent { label 'dual' } // Could run on a different agent
                    steps {
                        dir('starbucks') {
                            sh 'echo "Building Starbucks static content"'
                            archiveArtifacts artifacts: '**/*', fingerprint: true
                        }
                    }
                }
                stage('Build NodeJS') {
                    agent { label 'dual' } // Could run on a different agent
                    steps {
                        dir('nodejs') {
                            script {
                                docker.image('node:18-alpine').inside {
                                    sh 'npm install'
                                    sh 'npm run build'
                                }
                            }
                            archiveArtifacts artifacts: 'dist/**/*, package.json, package-lock.json', fingerprint: true
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Netflix Image') {
                    agent { label 'dual' }
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
                    agent { label 'dual' }
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
                    agent { label 'dual' }
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

        // stage('Archive Docker Images') {
        //     agent { label 'dual' }
        //     steps {
        //         script {
        //             sh """
        //                 docker save -o netflix-${IMAGE_TAG}.tar netflix:${IMAGE_TAG}
        //                 docker save -o starbucks-${IMAGE_TAG}.tar starbucks:${IMAGE_TAG}
        //                 docker save -o nodejs-${IMAGE_TAG}.tar nodejs:${IMAGE_TAG}
        //             """
        //             archiveArtifacts artifacts: '*.tar', fingerprint: true
        //         }
        //     }
        // }

        stage('Deploy to Docker') {
            agent { label 'dual' }
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
            agent { label 'dual' }
            steps {
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
}