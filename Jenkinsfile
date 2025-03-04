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
                    
                    echo "🔧 Installing Required Packages..."
                    sudo apt update -y
                    sudo apt install -y python3-pip unzip curl

                    echo "🐍 Upgrading Pip..."
                    pip3 install --upgrade pip

                    echo "📦 Installing Python Dependencies..."
                    pip3 install -r lambda-app/tests/requirements.txt
                    
                    echo "🔧 Installing pytest globally..."
                    pip3 install pytest  # Ensure pytest is installed globally

                    echo "☁️ Checking & Installing AWS SAM CLI..."
                    if ! command -v sam &> /dev/null
                    then
                        echo "🚀 Installing AWS SAM CLI..."
                        curl -Lo aws-sam-cli.zip https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
                        unzip -o aws-sam-cli.zip -d sam-installation  # Force overwrite
                        sudo ./sam-installation/install --update
                        rm -rf aws-sam-cli.zip sam-installation
                    else
                        echo "✅ AWS SAM CLI is already installed."
                    fi

                    echo "🔄 Setup Completed Successfully!"
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    echo "🧪 Running Tests..."
                    python3 -m pytest  # Use Python interpreter to run pytest
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    echo "🏗️ Building SAM Application..."
                    sam build -t lambda-app/template.yaml
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e
                    echo "🚀 Deploying SAM Application..."
                    sam deploy -t lambda-app/template.yaml --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
    }
}
