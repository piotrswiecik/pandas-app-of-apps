def frontendImage = "capricornpl/pandas-frontend"
def backendImage = "capricornpl/pandas-backend"
def dockerRegistry = ""
def registryCredentials = "dockerhub"
def backendDockerTag
def frontendDockerTag

pipeline {
    agent {
        label "agent"
    }

    tools {
        terraform 'Terraform'
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

        stage('adjust version') {
            steps {
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    withEnv([
                        "FRONTEND_IMAGE=$frontendImage:$frontendDockerTag",
                        "BACKEND_IMAGE=$backendImage:$backendDockerTag"
                    ]) {
                        docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
        }

        stage('selenium') {
            steps {
                sh "pip3 install -r test/selenium/requirements.txt"
                sh "python3 -m pytest test/selenium/frontendTest.py"
            }
        }

        stage('terraform') {
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/Panda-Academy-Core-2-0/Terraform'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init && terraform apply -auto-approve -var-file="terraform.tfvars"'
                    } 
                }
            }
        }

        stage('ansible') {
               steps {
                   script {
                        sh "ansible-galaxy install -r requirements.yml"
                        withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                                 "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                            ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                        }
                   }
               }
        }
        post {
            always {
                sh "docker-compose down"
                cleanWs()
            }
        }
    }
}
