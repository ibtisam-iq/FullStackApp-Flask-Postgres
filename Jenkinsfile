pipeline {
    agent any
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/jaiswaladi246/Multi-Tier-
Python-Postgres.git'
            }
        }
        stage('Setup Virtual Environment') {
            steps {
                sh '''
                    rm -rf venv # Remove any existing virtual environments
                    python3 -m venv venv # Create a new virtual environment
                    chmod -R 755 venv
                    bash -c "
                        source venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    "
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    bash -c "
                        source venv/bin/activate
                        pytest --cov=app --cov-report=xml
                        pytest --cov=app --cov-report=term-missing --disable-warnings
                    "
                '''
            }
        }
        stage('Sonar') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Python-project \
                        -Dsonar.projectName=Python-project \
                        -Dsonar.exclusions=venv/** \
                        -Dsonar.sources=. \
                        -Dsonar.python.coverage.reportPaths=coverage.xml'''
                }
            }
        }
    }
}
