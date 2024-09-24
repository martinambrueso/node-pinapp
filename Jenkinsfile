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
                - name: trivy
                  image: aquasec/trivy:latest
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
                }
            }
        }
        stage('Build image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'docker_auth', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin
                        '''
                    }

                    sh 'docker build -t tokn1/node_test:latest .'
                    sh 'docker tag tokn1/node_test:latest tokn1/node_test:latest'
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                container('trivy') {
                    sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL tokn1/node_test:latest || true'
                    // cambiar a --exit-code 1 para que falle si hay vulnerabilidades
                }
            }
        }
        stage('Push image') {
            steps {
                container('docker') {
                    sh 'docker push tokn1/node_test:latest'
                }
            }
        }
        stage('Deploy') {
            steps {
                container('node') {
                    sh 'kubectl apply -f k8s/deployment.yml'
                    sh 'kubectl apply -f k8s/service.yml'
                    sh 'kubectl rollout restart deployment pinapp-node-test'
                    sh 'kubectl get pods'
                }
            }
        }
    }
}
