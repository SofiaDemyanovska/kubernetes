pipeline {
    agent {
      node {
        label 'slave3'
      }
    }
    
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh '''
                docker build ./backend-dubbing-project  -t sofdem/back:${BUILD_NUMBER}
                docker build ./front-dubbing-project -t sofdem/front:${BUILD_NUMBER}
                '''
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'DockerHubCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh "docker login -u ${username} -p ${password}"
                    sh "docker push sofdem/back:${BUILD_NUMBER}"
                    sh "docker push sofdem/front:${BUILD_NUMBER}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh '''
                sed -i "s/tagVersion/${BUILD_NUMBER}/g" k8s.yml 
                '''
                sh "sshpass -p P@ssw0rd scp -o StrictHostKeyChecking=no k8s.yml root@10.26.3.198:/root/"
                script{
                    try{
                        sh "sshpass -p P@ssw0rd ssh root@10.26.3.198 kubectl apply -f ."
                    }catch(error){
                        sh "sshpass -p P@ssw0rd ssh root@10.26.3.198 kubectl create -f ."
                    }
                }
           }
       }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
