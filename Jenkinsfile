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
                script {
                    echo "Cloning Github repo to Jenkins............"
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/Sanikaaher/Hospitality-Revenue-Optimizer.git']])
                }
            }
        }

        stage('Build and Push Docker Image to GCR') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo "Building and Pushing Docker Image to GCR............."
                        sh """
                        # Authenticate gcloud
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud config set project ${GCP_PROJECT}
                        
                        # Configure Docker to use gcloud credentials for GCR
                        ${GCLOUD_BIN}/gcloud auth configure-docker gcr.io --quiet
                        
                        # Explicitly login Docker using access token (THIS IS THE KEY FIX)
                        TOKEN=\$(${GCLOUD_BIN}/gcloud auth print-access-token)
                        echo "\${TOKEN}" | ${DOCKER_BIN}/docker login -u oauth2accesstoken --password-stdin https://gcr.io
                        
                        # Build Docker image (will use cache from previous successful build)
                        ${DOCKER_BIN}/docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .
                        
                        # Push to GCR (should work now with proper authentication)
                        ${DOCKER_BIN}/docker push gcr.io/${GCP_PROJECT}/ml-project:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo "Deploying to Google Cloud Run............."
                        sh """
                        # Authenticate again for deployment
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud config set project ${GCP_PROJECT}
                        
                        # Deploy to Cloud Run
                        ${GCLOUD_BIN}/gcloud run deploy ml-project \\
                            --image=gcr.io/${GCP_PROJECT}/ml-project:latest \\
                            --platform=managed \\
                            --region=us-central1 \\
                            --allow-unauthenticated \\
                            --memory=2Gi \\
                            --cpu=2 \\
                            --timeout=300
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
            echo "Your ML model is now deployed and running on Cloud Run"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above for details."
        }
    }
}