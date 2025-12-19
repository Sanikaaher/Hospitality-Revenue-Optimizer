pipeline{
    agent any

    environment {
        VENV_DIR = 'venv'
        // GCP_PROJECT = "mlops-new-447207"
        // GCLOUD_PATH = "/var/jenkins_home/google-cloud-sdk/bin"
    }
    stages{
        stage('Clonig Github Repo to Jenkins'){
            steps{
                script{
                    echo'Clonig Github repo to jenkins..............'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/Sanikaaher/Hospitality-Revenue-Optimizer.git']])
                }
            }
        }

    stage('Setting up our Virtual Environment and Installing dependancies'){
            steps{
                script{
                    echo'Setting up our Virtual Environment and Installing dependancies.............'
                     sh '''
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    '''
                  
                }
            }
        }
    }
}
