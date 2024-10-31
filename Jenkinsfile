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
                        sh """
                        # Start SSH agent and add the private key
                        eval "$(ssh-agent -s)"
                        ssh-add ${SSH_KEY}

                        # Add GitHub to known hosts to avoid host verification errors
                        mkdir -p ~/.ssh
                        ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

                        # Clone the repository and update the image tag
                        git clone git@github.com:ankur-devops-demo/html-demo-app.git
                        cd html-demo-app

                        # Use sed to update the newTag in kustomization.yaml
                        sed -i '/name: ankurnema\\/html-app/{n;s/newTag: .*/newTag: ${params.VERSION}/}' kustomization.yaml

                        # Commit and push changes
                        git add kustomization.yaml
                        git commit -m "Update image tag to version ${params.VERSION}"
                        git push origin main

                        # Stop SSH agent after use
                        ssh-agent -k
                        """
                    }
                }
            }
        }
    }
}