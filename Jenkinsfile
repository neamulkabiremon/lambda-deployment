pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {

        stage('Setup') {
            steps {
                sh '''
                    set -e
                    python3 -m venv venv
                    . venv/bin/activate
                    pip3 install --upgrade pip
                    pip3 install -r lambda-app/tests/requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    . venv/bin/activate
                    sam build -t lambda-app/template.yaml
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e
                    . venv/bin/activate
                    sam deploy -t lambda-app/template.yaml --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
    }
}
