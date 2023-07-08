pipeline {
    agent any
    environment {
        TARGET_REGION = "us-east-1"
        BOT_URL = credentials('telegram')
    }    
    stages {
        stage('Install dependencies ') {
            parallel {
                stage('Init') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        args '-u root:root'
                    }           
                 }
                    steps {
                        sh 'npm install'
                    }
                }
                stage('Test') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        args '-u root:root'
                    }           
                 }
                    steps {
                        sh 'npm run test'
                    }
                }
            } // end parallel
        }

        stage('Build-Docker-ECR') {
            steps {
                withAWS(credentials: 'aws-roxsross', region: "${TARGET_REGION}" ) {
                   sh './automation/docker_build.sh'
                   sh './automation/docker_push.sh'  
                }     
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploy"
            }
        }
        
        stage('Notifications') {
            steps {
                echo "notify"
            }
        }
    }
}