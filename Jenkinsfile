pipeline {
    agent {
        label "agent"
    }

    tools {
        terraform 'terraform'
    }

    parameters {
        string(name: 'backendDockerTag', defaultValue: '', description: 'Backend Docker image tag')
        string(name: 'frontendDockerTag', defaultValue: '', description: 'Frontend docker image tag')
    }

    stages {
        stage('clone') {
            steps {
                checkout scm
            }
        }
        
        stage('container clean') {
            steps {
                sh 'docker rm -f frontend backend'
            }
        }

        def backendDockerTag
        def frontendDockerTag

        stage('adjust version') {
            steps {
                script {
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }

        def frontendImage = "capricornpl/pandas-frontend"
        def backendImage = "capricornpl/pandas-backend"
        def dockerRegistry = ""
        def registryCredentials = "dockerhub"

        stage('deploy') {
            steps {
                script {
                    withEnv([
                        "FRONTEND_IMAGE=$frontendImage:$frontendDockerTag",
                        "BACKEND_IMAGE=$backendImage:$backendDockerTag"
                    ]) {
                        docker.withRegistry("$$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
        }

    }
}