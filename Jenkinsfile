pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git(branch: 'main', credentialsId: '07a0c530-771f-482f-9d7e-fcdea96fcc78', url: 'https://github.com/borisK0/simple-website.git')
      }
    }

    stage('Authenticate to GCP') {
      steps {
        withCredentials(bindings: [file(credentialsId: '65a7cfe4-8768-42ca-8f10-97454823238f', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud config set project $PROJECT_ID
                    '''
        }

      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
                docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:latest .
                '''
      }
    }

    stage('Push to Artifact Registry') {
      steps {
        sh '''
                gcloud auth configure-docker $REGION-docker.pkg.dev --quiet
                docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:latest
                '''
      }
    }
####
  }
  environment {
    PROJECT_ID = 'boris-temp-for-lab'
    REPO = 'my-repo'
    REGION = 'europe-west1'
    IMAGE_NAME = 'my-js-app'
  }
}

stage('Deploy Container') {
    steps {
        sh '''
        # Stop old container if it exists
        docker rm -f my-js-app || true

        # Pull latest image from Artifact Registry
        docker pull $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:latest

        # Run new container on host port 3000 (mapping to container port 80)
        docker run -d --name my-js-app -p 3000:80 $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:latest
        '''
    }
}
