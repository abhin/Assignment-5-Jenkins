def buildTag = ''

def buildDockerImage(tag) {
    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        try {
            sh """
                docker build -t assment5app:${tag} .
                echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                docker tag assment5app:${tag} ${DOCKER_USER}/assment5app:${tag}
                docker push ${DOCKER_USER}/assment5app:${tag}
            """
        } catch (Exception e) {
            error "Docker commands failed: ${e}"
        }
    }
}

pipeline {
    agent { label 'Built-In Node' }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    def date = new Date().format('yyyyMMdd')
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    currentBuild.displayName = buildTag
                }
            }
        }

        stage('Use Tag') {
            steps {
                script {
                    echo "The build tag is: ${buildTag}"
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
                script {
                    buildDockerImage(buildTag)
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        helm upgrade --install assment5app ./helm-chart \
                        --set image.tag=${buildTag} \
                        --set image.repository=${DOCKER_USER}/assment5app
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'All images built, pushed, and app deployed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
