pipeline {
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.'
    }

    stages {
        stage('Prepare') {
            agent any

            steps {
                echo "Clonning Repository"

                git url: "https://github.com/ojj1123/cicd-study.git",
                branch: 'maintain',
                credentialsId: 'git-test-jenkins'
            }

            post {
                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                    echo "i tried"
                }

                cleanup {
                    echo "after all other post condition"
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo 'Deploying Frontend'
                dir ('./client') {
                    sh '''
                        aws s3 sync ./ s3://jeongjin
                    '''
                }
            }
            // 프론트엔드 작업을 여기서 처리함
            post {
                success {
                    echo 'Successfully cloned Repository'

                    mail to: 'rojay.developer@gmail.com',
                    subject: 'Deploying Frontend Success',
                    body: 'Successfully deployed frontend!'
                }
                
                failure {
                    echo 'I filed :('

                    mail to: 'rojay.developer@gmail.com',
                    subject: 'Failed Pipeline',
                    body: 'Something is wrong deploy frontend!'
                }

            }
        }

        stage('Lint Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }
            
            steps {
                dir ('./server'){
                    sh '''
                    npm install&&
                    npm run lint
                    '''
                }
            }
        }

        stage('Test Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }
            steps {
                echo 'Test Backend'

                dir ('./server'){
                    sh '''
                    npm install
                    npm run test
                    '''
                }
            }
        }

        stage('Build Backend') {
            agent any
            steps {
                echo 'Build Backend'

                dir ('./server'){
                    sh """
                    docker build . -t server --build-arg env=${PROD}
                    """
                }
            }

            post {
                failure {
                    error 'This pipeline stops here...'
                }
            }
        }

        stage('Deploy Backend') {
            agent any

            steps {
                echo 'Build Backend'

                dir ('./server'){
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                }
            }

            post {
                success {
                    mail  to: 'rojay.developer@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                }
            }
        }
    }
}