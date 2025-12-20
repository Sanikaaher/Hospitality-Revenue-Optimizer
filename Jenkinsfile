pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = "alpine-realm-479408-j0"
        GCLOUD_BIN = "/usr/bin"
        DOCKER_BIN = "/usr/bin"
    }

    stages {
        stage('Cloning Github repo') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/Sanikaaher/Hospitality-Revenue-Optimizer.git']])
            }
        }

        stage('Build and Push') {
            steps {
                // Use the credential ID from your screenshot
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh """
                        # Authenticate
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud auth configure-docker --quiet

                        # Build using Cache (This skips the 18-minute wait)
                        ${DOCKER_BIN}/docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .
                        
                        # Push (This will be instant since we already pushed it)
                        ${DOCKER_BIN}/docker push gcr.io/${GCP_PROJECT}/ml-project:latest 
                        """
                    }
                }
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh """
                        ${GCLOUD_BIN}/gcloud run deploy ml-project \\
                            --image=gcr.io/${GCP_PROJECT}/ml-project:latest \\
                            --platform=managed \\
                            --region=us-central1 \\
                            --allow-unauthenticated
                        """
                    }
                }
            }
        }
    }
}