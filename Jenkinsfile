pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
                containers:
                - name: node
                  image: tokn1/node_custom:latest
                  command:
                  - cat
                  tty: true
                - name: docker
                  image: docker:19.03.12
                  command:
                  - cat
                  tty: true
                  volumeMounts:
                  - name: docker-sock
                    mountPath: /var/run/docker.sock
                volumes:
                - name: docker-sock
                  hostPath:
                    path: /var/run/docker.sock
            '''
        }
    }
    stages {
        stage('Clone') {
            steps {
                container('node') {
                    git branch: 'main', url: 'https://github.com/martinambrueso/node-pinapp.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
            }
        }
        stage('Run Docker') {
            steps {
                container('docker') {
                    sh 'docker -v'
                    
                    withCredentials([usernamePassword(credentialsId: 'docker_auth', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin
                        '''
                    }

                    sh 'docker build -t tokn1/node_test:latest .'
                    sh 'docker tag tokn1/node_test:latest tokn1/node_test:latest'

                    sh 'docker push tokn1/node_test:latest'
                }
            }
        }
        stage('Run Node.js') {
            steps {
                container('node') {
                    sh 'node -v'
                    sh 'ls -la'
                    sh 'pwd'
                    sh 'kubectl apply -f k8s/deployment.yml'
                    sh 'kubectl apply -f k8s/service.yml'
                    sh 'kubectl get pods'
                }
            }
        }
    }
}