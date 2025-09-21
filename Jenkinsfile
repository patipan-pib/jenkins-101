pipeline {
    agent any
    environment {
        REGISTRY = 'your-registry.io'
        IMAGE = "${REGISTRY}/simple-api:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:YOUR-ORG/simple-api.git'
            }
        }

        stage('Unit Test (VM2)') {
            steps {
                sshagent(credentials: ['ssh-vm2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no user@VM2 "
                        rm -rf ~/simple-api && \
                        git clone https://github.com/YOUR-ORG/simple-api.git && \
                        cd simple-api && pytest -q
                    "
                    '''
                }
            }
        }

        stage('Build & Push Docker (VM2)') {
            steps {
                sshagent(credentials: ['ssh-vm2']) {
                    sh '''
                    ssh user@VM2 "
                        cd ~/simple-api && \
                        docker build -t ${IMAGE} . && \
                        docker login -u USER -p PASS ${REGISTRY} && \
                        docker push ${IMAGE}
                    "
                    '''
                }
            }
        }

        stage('Robot Test (VM2)') {
            steps {
                sshagent(credentials: ['ssh-vm2']) {
                    sh '''
                    ssh user@VM2 "
                        rm -rf ~/simple-api-robot && \
                        git clone https://github.com/YOUR-ORG/simple-api-robot.git && \
                        cd simple-api-robot && robot tests/
                    "
                    '''
                }
            }
        }

        stage('Deploy (VM3)') {
            steps {
                sshagent(credentials: ['ssh-vm3']) {
                    sh '''
                    ssh user@VM3 "
                        docker pull ${IMAGE} && \
                        docker rm -f simple-api || true && \
                        docker run -d --name simple-api -p 8080:8080 ${IMAGE}
                    "
                    '''
                }
            }
        }
    }
}
