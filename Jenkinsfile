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
                    # Create a virtual environment
                    python3 -m venv venv
                    . venv/bin/activate

                    # Upgrade pip and install dependencies
                    pip3 install --upgrade pip
                    pip3 install -r lambda-app/tests/requirements.txt

                    # Install SAM CLI if missing
                    if ! command -v sam &> /dev/null; then
                        echo "SAM CLI not found! Installing..."
                        curl -L https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip -o aws-sam-cli.zip
                        unzip aws-sam-cli.zip -d sam-installation
                        sudo ./sam-installation/install
                        rm -rf aws-sam-cli.zip sam-installation
                    fi

                    # Verify SAM installation
                    sam --version
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    pytest
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    sam build -t lambda-app/template.yaml
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e
                    sam deploy -t lambda-app/template.yaml --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
    }
}
