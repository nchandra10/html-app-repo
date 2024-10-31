pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
    }
    environment {
        SSH_KEY_CREDENTIALS = 'git-ssh-credentials' // Jenkins SSH key credential ID for Git
        DOCKER_CREDENTIALS = 'docker-hub-credentials' // Docker Hub credential ID
    }
    stages {
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        docker.build("ankurnema/html-app:${params.VERSION}")
                    }
                }

            }
        }
        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', env.DOCKER_CREDENTIALS) {
                            docker.image("ankurnema/html-app:${params.VERSION}").push()
                        }
                    }
                }
            }
        }
        stage('Update Git Repository') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: env.SSH_KEY_CREDENTIALS, keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                        # Start SSH agent and add the private key
                        eval "$(ssh-agent -s)"
                        ssh-add $SSH_KEY

                        # Clone the repository and update the image tag
                        git clone git@github.com:ankur-devops-demo//html-demo-app.git
                        cd k8s-manifest-repo

                        # Use yq to update the image tag in kustomization.yaml
                        yq eval '.images[] |= select(.name == "ankurnema/html-app").newTag = "'${params.VERSION}'"' -i kustomization.yaml

                        # Commit and push changes
                        git add kustomization.yaml
                        git commit -m "Update image tag to version ${params.VERSION}"
                        git push origin main

                        # Stop SSH agent after use
                        killall ssh-agent
                        '''
                    }
                }
            }
        }
    }
}