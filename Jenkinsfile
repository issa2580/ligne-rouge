pipeline {
    environment {
        webDockerImageName = "martinez42/ligne-rouge-web"
        dbDockerImageName = "martinez42/ligne-rouge-db"
        webDockerImage = ""
        dbDockerImage = ""
        registryCredential = 'docker-credentiel'
        KUBECONFIG = "/home/rootkit/.kube/config"
        TERRA_DIR  = "/home/rootkit/ligne-rouge/terraform"
        ANSIBLE_DIR = "/home/rootkit/ligne-rouge/ansible"
    }
    agent any
    stages {
        stage('Checkout Source') {
            steps {
                git 'https://github.com/issa2580/ligne-rouge.git'
            }
        }
        stage('Build Web Docker image') {
            steps {
                script {
                    webDockerImage = docker.build webDockerImageName, "-f App.Dockerfile ."
                }
            }
        }
        stage('Build DB Docker image') {
            steps {
                script {
                    dbDockerImage = docker.build dbDockerImageName, "-f Db.Dockerfile ."
                }
            }
        }
        stage('Pushing Images to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        webDockerImage.push('latest')
                        dbDockerImage.push('latest')
                    }
                }
            }
        }
        stage('Set Permissions for Kubernetes Config') {
            steps {
                script {
                    sh """
                    sudo chown jenkins:jenkins ${KUBECONFIG}
                    sudo chmod 777 ${KUBECONFIG}
                    """
                }
            }
        }
        stage("Provision Kubernetes Cluster") {
            steps {
                script {
                    sh """
                    sudo chown -R jenkins:jenkins ${TERRA_DIR}
                    sudo chmod -R 777 ${TERRA_DIR}
                    cd ${TERRA_DIR}
                    terraform init
                    terraform plan
                    terraform apply --auto-approve
                    """
                }
            }
        }
        stage('Install Python dependencies') {
            steps {
                withPythonEnv('/usr/bin/python3') {
                    sh """
                    pip install kubernetes
                    """
                }
            }
        }
        stage('Deploying with Ansible') {
            steps {
                ansiblePlaybook(
                    installation: 'Ansible',
                    playbook: "${ANSIBLE_DIR}/playbook.yml",
                    inventory: 'localhost,',
                    extras: '--extra-vars "ansible_python_interpreter=/usr/bin/python3"'
                )
            }
        }   
    }
    post {
        success {
            slackSend channel: 'groupe4', message: 'Success to deploy'
        }
        failure {
            slackSend channel: 'groupe4', message: 'Failed to deploy'
        }
    }
}