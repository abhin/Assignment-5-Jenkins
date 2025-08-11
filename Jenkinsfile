def buildTag = ''

pipeline {
    agent {
        kubernetes {
            label 'kaniko-build-pod'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args:
      - --cache=true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred  # Kubernetes secret with Docker credentials
"""
        }
    }

    environment {
        DOCKER_USER = '' // Will be set from credentials below
        // DOCKER_PASS not needed explicitly here, kaniko uses mounted secret
    }

    stages {
        stage('Check Node & Docker') {
            steps {
                sh '''
                    echo "Running on node: $(hostname)"
                    echo "Docker location: $(which docker || echo 'docker not found')"
                    docker --version || echo 'docker command not available'
                '''
            }
        }

        stage('Generate Tag') {
            steps {
                script {
                    def date = new Date().format('yyyyMMdd')
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    currentBuild.displayName = buildTag
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/abhin/Assignment-5-Jenkins.git', branch: 'master'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    container('kaniko') {
                        script {
                            // Set DOCKER_USER env var for image tagging
                            env.DOCKER_USER = DOCKER_USER
                            sh """
                            /kaniko/executor \\
                              --dockerfile=Dockerfile \\
                              --context=dir://\$(pwd) \\
                              --destination=${DOCKER_USER}/assment5app:${buildTag} \\
                              --cache=true
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        helm upgrade --install assment5app ./helm-chart \\
                        --set image.tag=${buildTag} \\
                        --set image.repository=${DOCKER_USER}/assment5app
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build, push, and deploy succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
