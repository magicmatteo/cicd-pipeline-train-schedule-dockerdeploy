pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'example-solution'
            }
            steps {
                script {
                    app = docker.build('golfplease/train-schedule')
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'example-solution'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'example-solution'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([sshUserPrivateKey(credentialsId: 'docker-ssh', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                    script {
                        sh "ssh -o StrictHostKeyChecking=no -i $KEYFILE $USERNAME@$prod_ip \"docker pull golfplease/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no -i $KEYFILE $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no -i $KEYFILE $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no -i $KEYFILE $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8082 -d golfplease/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}