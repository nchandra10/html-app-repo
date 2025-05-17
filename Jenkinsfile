pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
    }
    environment {
        GIT_CREDENTIALS = 'git-ssh-credentials' // Jenkins username/password credential ID for Git (renamed for clarity)
        DOCKER_CREDENTIALS = 'docker-hub-credentials' // Docker Hub credential ID
    }
    stages {
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        docker.build("nitincc10/html-app:${params.VERSION}")
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', env.DOCKER_CREDENTIALS) {
                            docker.image("nitincc10/html-app:${params.VERSION}").push()
                        }
                    }
                }
            }
        }
        stage('Update Git Repository') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                        # Clone the repository using HTTPS with embedded credentials
                        git clone "https://\$GIT_USERNAME:\$GIT_PASSWORD@github.com/nchandra10/html-demo-app.git"
                        cd html-demo-app

                        # Use sed to update the newTag in kustomization.yaml
                        sed -i '/name: nchandra10\\/html-app/{n;s/newTag: .*/newTag: ${params.VERSION}/}' kustomization.yaml

                        # Set Git user configuration
                        git config user.email "automation@users.noreply.github.com"
                        git config user.name "Automation User"

                        # Commit and push changes using HTTPS with embedded credentials
                        git add kustomization.yaml
                        git commit -m "Update image tag to version ${params.VERSION}"
                        git push "https://\$GIT_USERNAME:\$GIT_PASSWORD@github.com/nchandra10/html-demo-app.git" main
                        """
                    }
                }
            }
        }
    }
}
