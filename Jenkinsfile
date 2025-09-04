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
        stage(Checkout) {
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
        stage(Build_Docker_Images) {
            parallel {
                stage(Netflix) {
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
                stage(Starbucks) {
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
                stage(Nodejs) {
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
        stage(Deploy_to_Docker) {
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
            echo "Deployment completed. Applications are running:"
            echo "Netflix: http://${AGENT_PUBLIC_IP}:8081"
            echo "StarBucks: http://${AGENT_PUBLIC_IP}:8082"
            echo "Node JS: http://${AGENT_PUBLIC_IP}:3000"
        }
    }
}
