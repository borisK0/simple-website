pipeline {
    agent any

    environment {
        PROJECT_ID = "boris-temp-for-lab"
        REPO = "my-repo"
        REGION = "europe-west1"
        IMAGE_NAME = "my-js-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/borisK0/simple-website.git'
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: '65a7cfe4-8768-42ca-8f10-97454823238f', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
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
                # Create a Docker image for a simple static site
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
    }
}
