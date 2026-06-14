pipeline {
    agent any
    tools {
        nodejs "my-nodejs"
    }
    environment {
        DOCKER_USER = "emrearabacioglu"
        IMAGE_NAME = "${DOCKER_USER}/nodejs-app:1.0.${BUILD_NUMBER}"
    }
    stages {
        stage('increment version') {
            when { 
                not { changelog '.*ci: version bump.*' } 
            }
            steps {
                dir('app') {
                    sh 'npm version patch'
                }
            }
        }
        stage('Run tests') {
            when { 
                not { changelog '.*ci: version bump.*' } 
            }
            steps {
                dir('app') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('Build and Push docker image') {
            when { 
                allOf {
                    branch 'main'
                    not { changelog '.*ci: version bump.*' }
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "docker build -t ${IMAGE_NAME} ."
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }
        stage('deploy to EC2') {
            when { 
                allOf {
                    branch 'main'
                    not { changelog '.*ci: version bump.*' }
                }
            }
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
            when { 
                allOf {
                    branch 'main'
                    not { changelog '.*ci: version bump.*' }
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'git config --global user.email "jenkins@example.com"'
                    sh 'git config --global user.name "Jenkins"'
                    sh 'git add app/package.json app/package-lock.json'
                    sh 'git commit -m "ci: version bump"'
                    sh "git push https://${USER}:${PASS}@github.com/emrearabacioglu/aws-exercises HEAD:main"
                }
            }
        }
    }     
}
