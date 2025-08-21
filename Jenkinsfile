pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
    }
    environment {
        SSH_KEY_CREDENTIALS = 'git-ssh-credentials'
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        docker.build("vamckumar/html-app:${params.VERSION}")
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', env.DOCKER_CREDENTIALS) {
                            docker.image("vamckumar/html-app:${params.VERSION}").push()
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
                        eval "\$(ssh-agent -s)"
                        ssh-add \$SSH_KEY

                        mkdir -p ~/.ssh
                        ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

                        git clone git@github.com:VAMCKUMAR/html-demo-app.git
                        cd html-demo-app

                        sed -i '/name: vamckumar\\/html-app/{n;s/newTag: .*/newTag: ${params.VERSION}/}' kustomization.yaml

                        git config user.email "automation@users.noreply.github.com"
                        git config user.name "Automation User"

                        git add kustomization.yaml
                        git commit -m "Update image tag to version ${params.VERSION}"
                        git push origin main

                        ssh-agent -k
                        """
                    }
                }
            }
        }
    }
}
