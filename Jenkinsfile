def buildTag = ''

def buildDockerImage(tag) {
    withCredentials([usernamePassword(credentialsId: 'docker-credentails', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        sh """
            docker build -t assnmnt5jenkins:${tag} .
            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
            docker tag assnmnt5jenkins:${tag} ${DOCKER_USER}/assnmnt5jenkins:${tag}
            docker push ${DOCKER_USER}/assnmnt5jenkins:${tag}
        """
    }
}

pipeline {
    agent { label 'built_agent' }

    parameters {
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace to deploy to')
    }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    def date = new Date().format('yyyyMMdd')
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    env.BUILD_TAG = buildTag
                    currentBuild.displayName = buildTag
                }
            }
        }

        stage('Show Tag') {
            steps {
                echo "The build tag is: ${env.BUILD_TAG}"
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
                    buildDockerImage(env.BUILD_TAG)
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    input message: "Do you want to proceed with Kubernetes deployment?", ok: 'Deploy'
                }

                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
                    script {
                        sh """
                            sed "s/IMAGE_TAG/${env.BUILD_TAG}/g" deployment.yaml > deployment-temp.yaml
                            kubectl apply -f deployment-temp.yaml -n ${params.NAMESPACE}
                            rm -f deployment-temp.yaml
                        """
                    }
                }
            }
        }
    }
}
