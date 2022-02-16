pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "latest"
    STAGING = "eazytraining-staging-staging"
    PRODUCTION = "eazytraining-production-prod"
  }
  agent none
  stages {
      stage('Build image') {
          agent any
          steps {
             script {
               sh 'docker build -t fdamlabin/${IMAGE_NAME}:${IMAGE_TAG} .'
             }
          }
      }
    
      stage('Run container on builded image') {
          agent any
          steps {
             script {
               sh '''
                  docker run -d -p 8085:5000 -e PORT=5000 --name ${IMAGE_NAME} fdamlabin/${IMAGE_NAME}:${IMAGE_TAG}
                  sleep 10
               '''
             }
          }
      }
    
      stage('Test image') {
          agent any
          steps {
             script {
               sh '''
                  curl http://192.168.56.5:8085 | grep -q "Hello world!"
               '''
             }
          }
      }
    
      stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
                  docker stop $IMAGE_NAME
                  docker rm -f $IMAGE_NAME
               '''
             }
          }
      }
    
      stage('Push image in staging and deploy') {
          when {
            expression { GIT_BRANCH == 'origin/master' } 
          }
          agent any
          environment { 
            HEROKU_API_KEY = credentials('heroku_api_key')
          }
          steps {
             script {
               sh '''
                  heroku container:login
                  heroku create $STAGING || echo "project already exist"
                  heroku container:push -a $STAGING web
                  heroku container:release -a $STAGING web
               '''
             }
          }
      }
    
      stage('Push image in prod and deploy') {
          when {
            expression { GIT_BRANCH == 'origin/master' } 
          }
          agent any
          environment { 
            HEROKU_API_KEY = credentials('heroku_api_key')
          }
          steps {
             script {
               sh '''
                  heroku container:login
                  heroku create $PRODUCTION || echo "project already exist"
                  heroku container:push -a $PRODUCTION web
                  heroku container:release -a $PRODUCTION web
               '''
             }
          }
      }
  }
  post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [{env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [{env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }
}
