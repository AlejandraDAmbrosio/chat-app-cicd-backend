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

        stage('Build') {
            steps {
                echo "build"
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