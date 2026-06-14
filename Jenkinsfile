pipeline {
    agent any
      tools {
        nodejs "my-nodejs"
      }
      stages {
        stage('increment version') {
          ...
        }
        stage('Run tests') {
          ...
        }
        stage('Build and Push docker image') {
          ...
        }
        stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@63.186.60.132"

                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }     
                }
            }
        }
        stage('commit version update') {
          ...
        }
    }     
}

