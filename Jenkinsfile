pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = "alpine-realm-479408-j0"
        // We tell Jenkins exactly where the tools are hiding
        GCLOUD_BIN = "/root/google-cloud-sdk/bin"
        DOCKER_BIN = "/usr/local/bin"
    }

    stages {
        stage('Cloning Github repo to Jenkins') {
            steps {
                script {
                    echo 'Cloning Github repo to Jenkins............'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/Sanikaaher/Hospitality-Revenue-Optimizer.git']])
                }
            }
        }

        stage('Setting up our Virtual Environment and Installing dependancies') {
            steps {
                script {
                    echo 'Setting up our Virtual Environment and Installing dependancies............'
                    sh '''
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    '''
                }
            }
        }

        stage('Building and Pushing Docker Image to GCR') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo 'Building and Pushing Docker Image to GCR.............'
                        sh """
                        # Use absolute paths to bypass WSL environment errors
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud config set project ${GCP_PROJECT}
                        ${GCLOUD_BIN}/gcloud auth configure-docker --quiet

                        ${DOCKER_BIN}/docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .
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
                        echo 'Deploy to Google Cloud Run.............'
                        sh """
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud config set project ${GCP_PROJECT}

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