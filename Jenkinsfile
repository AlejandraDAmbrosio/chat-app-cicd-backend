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

        stage('sast') {
            parallel {
                stage('secrets') {
                    steps {
                        sh './automation/security.sh horusec'
                        stash name: 'report_horusec.json', includes: 'report_horusec.json'
                        }
                    }
                stage('Semgrep') {
                    agent{
                        docker{
                            image 'returntocorp/semgrep'
                            args '-u root:root'                    
                        }
                    }
                    steps {
                         sh '''
                        cat << 'EOF' | bash
                            semgrep ci --config=auto --json --output=report_semgrep.json --max-target-bytes=2MB
                            EXIT_CODE=$?
                            if [ "$EXIT_CODE" = "0" ] || [ "$EXIT_CODE" = "1" ]
                            then
                                exit 0
                            else
                                exit $EXIT_CODE
                            fi
                        EOF
                         '''
                         stash name: 'report_semgrep.json', includes: 'report_semgrep.json'
                    }
                }
            stage('audit') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        args '-u root:root'
                    }           
                 }
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh 'npm audit --registry=https://registry.npmjs.org -audit-level=critical --json > report_npmaudit.json'
                            stash name: 'report_npmaudit.json', includes: 'report_npmaudit.json'
                        } 
                    }
                }
        }
        stage('Build') {
            parallel {
            stage('update-compose') {
                steps {
                    sh './automation/aws_beanstalk.sh compose'  
                    }
                }
            stage('Build-Docker-ECR') {
                steps {
                    withAWS(credentials: 'aws-roxsross', region: "${TARGET_REGION}" ) {
                    sh './automation/docker_build.sh'
                    sh './automation/docker_push.sh'  
                        }     
                     }
                }
            }
        }

        stage('Check-AWS-Beanstalk') {
            steps {
                withAWS(credentials: 'aws-roxsross', region: "${TARGET_REGION}" ) {
                sh './automation/aws_beanstalk.sh check'
                }
            }
        }

        stage('Deploy-AWS-Beanstalk') {
            steps {
                withAWS(credentials: 'aws-roxsross', region: "${TARGET_REGION}" ) {
                sh './automation/aws_beanstalk.sh deploy'
                }
            }
        }
        
        stage('Notifications') {
            // when {
            //     branch 'master'
            // }
            steps {
                sh './automation/notification.sh'
            }
        }
    }
}