pipeline {
    agent any 
    environment {
        DOCKER_TAG = getDockerTag()
    }
    stages {
        stage('Build Docker image') {
            steps {
                sh "docker build . -t yakmandocker/nodeapp:${DOCKER_TAG}"
            }
        }
        stage('Push the Image to DockerHub registry') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u yakmandocker -p ${dockerHubPwd}"
                    sh "docker push yakmandocker/nodeapp:${DOCKER_TAG}"
            }
                
            }
            
        }
        stage('Deploy to K8S') {
            steps {
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@"K8s-bootstrap public ip":/home/ec2-user/"
                script {
                    try {
                        sh "ssh ec2-user@"K8s-bootstrap public ip" kubectl apply -f . -n nodejs"
                    }catch(error) {
                        sh "ssh ec2-user@"K8s-bootstrap public ip" kubectl create -f . -n nodejs"
                    }
                }
            }
            }
        }
    }

}

def getDockerTag() {
    def tag = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag 
}