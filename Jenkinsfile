pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = "alpine-realm-479408-j0"
        // Standard paths for the tools
        GCLOUD_BIN = "/usr/bin"
        DOCKER_BIN = "/usr/bin"
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
                    # Clean up any old broken environments
                    rm -rf ${VENV_DIR}
                    
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip setuptools wheel
                    
                    # FORCE FIX: These specific versions have pre-built files for Python 3.13.
                    # This stops the 500-line compilation errors immediately.
                    pip install "numpy>=2.1.0" "scikit-learn>=1.5.0" "pandas>=2.2.0"
                    
                    # Install your project but tell it NOT to try and "fix" the versions we just installed
                    pip install --no-deps -e .
                    
                    # Install remaining requirements
                    pip install flask google-cloud-storage pyyaml imbalanced-learn lightgbm mlflow
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
                        # Use absolute paths and force gcloud setup
                        ${GCLOUD_BIN}/gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        ${GCLOUD_BIN}/gcloud config set project ${GCP_PROJECT}
                        ${GCLOUD_BIN}/gcloud auth configure-docker --quiet

                        # The build command
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