```groovy
pipeline {
    agent any

    environment {
        IMAGE = "7760472594/python-demo:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Python') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "$IMAGE" .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )
                ]) {
                    sh '''
                        set -eu

                        echo "$DH_PASS" | docker login \
                            -u "$DH_USER" --password-stdin

                        docker push "$IMAGE"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-file',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        set -eu

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml

                        kubectl set image deployment/python-demo \
                            python-demo="$IMAGE"

                        kubectl rollout status \
                            deployment/python-demo \
                            --timeout=180s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-file',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        set -eu

                        kubectl get deployment python-demo
                        kubectl get pods -o wide
                        kubectl get service python-demo
                    '''
                }
            }
        }
    }
}
```
