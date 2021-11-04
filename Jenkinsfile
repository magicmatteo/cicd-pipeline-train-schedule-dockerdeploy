pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                echo prod_ip
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
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
            
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                        
                    }
                }
            }
        }
        stage('DeployToStaging') {
            
            steps {
                                
                withCredentials([sshUserPrivateKey(credentialsId: 'docker-ssh', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                    script {
                        def remote = [:]
                        remote.name = prod_ip
                        remote.host = prod_ip
                        remote.allowAnyHosts = true
                        
                        remote.user = USERNAME
                        remote.identityFile = KEYFILE
                        sshCommand remote: remote, command: "docker pull golfplease/train-schedule:${env.BUILD_NUMBER}", failOnError: 'true'
                        try {
                            sshCommand remote: remote, command: "docker stop train-schedule", failOnError: 'false'
                            sshCommand remote: remote, command: "docker rm train-schedule", failOnError: 'false'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sshCommand remote: remote, command: "docker run --restart always --name train-schedule -p 8082:8080 -d golfplease/train-schedule:${env.BUILD_NUMBER}", failOnError: 'true'
                    }
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                milestone(1)
                input 'Does staging look okay? Proceed to deploy to production'
                sh 'docker pull golfplease/train-schedule:latest'
                sh 'docker stop train-schedule-prod'
                sh 'docker rm train-schedule-prod'
                sh 'docker run --restart always --name train-schedule-prod -p 8083:8080 -d golfplease/train-schedule:latest'
            }
        }
    }
}