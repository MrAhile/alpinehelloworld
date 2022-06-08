pipeline{
    environement{
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "eazytraining-sataging"
        PRODUCTION = "eazytraining-production"
    }
    agen none
    stages{
        stage('Build image'){
            agent any
            steps{
                script{
                    sh 'docker build -t eazytraining/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on build image'){
            agent any
            steps{
                script{
                    sh '''
                        docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
                        sleep 10
                       '''
                }
            }
        }
        stage('Test image'){
            agent any
            steps{
                script{
                    sh '''
                        curl http://172.18.0.1 | grep -q "Hello world!"
                       '''
                }
            }
        }
        stage('Clean container'){
            agent any
            steps{
                script{
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm -f $IMAGE_NAME
                       '''
                }
            }
        }
        stage('Push image in staging deploy'){
            when{
                expression {GIT_BRANCH=='origin/master'}
            }
            agent any
            environement {
                HEROKU_API_KEY='credentials('heroku_api_key')'
            }
            steps{
                script{
                    sh  '''
                        heroku container:login
                        heroku create $STAGING || echo "project already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                        '''

                }
            }
        }
        stage('Push image in production deploy'){
            when{
                expression {GIT_BRANCH=='origin/master'}
            }
            agent any
            environement {
                HEROKU_API_KEY='credentials('heroku_api_key')'
            }
            steps{
                script{
                    sh  '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                        '''

                }
            }
        }
    }
}
