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
                    sudo apt install -y python3.9 python3.9-venv python3.9-dev unzip curl
                    
                    echo "🐍 Ensuring Python 3.9 is Default..."
                    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.9 1

                    echo "🐍 Upgrading Pip..."
                    python3.9 -m ensurepip --default-pip
                    python3.9 -m pip install --upgrade pip

                    echo "📦 Installing Python Dependencies..."
                    python3.9 -m pip install -r lambda-app/tests/requirements.txt
                    
                    echo "🔧 Installing pytest globally..."
                    python3.9 -m pip install pytest  # Ensure pytest is installed for Python 3.9

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
                    python3.9 -m pytest  # Use Python 3.9 explicitly
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -e
                    echo "🏗️ Building SAM Application..."
                    sam build -t lambda-app/template.yaml --use-container  # Ensures compatibility
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
