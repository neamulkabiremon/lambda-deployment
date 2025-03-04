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
                    sudo apt install -y lsb-release

                    UBUNTU_VERSION=$(lsb_release -rs)

                    if [ "$UBUNTU_VERSION" = "22.04" ]; then
                        echo "🐍 Installing Python 3.10 for Ubuntu 22.04..."
                        sudo apt install -y python3.10 python3.10-venv python3.10-dev python3-pip
                        PYTHON_BIN="python3.10"
                    else
                        echo "🐍 Installing Python 3.9 using Deadsnakes PPA..."
                        sudo add-apt-repository ppa:deadsnakes/ppa -y
                        sudo apt update -y
                        sudo apt install -y python3.9 python3.9-venv python3.9-dev python3-pip
                        PYTHON_BIN="python3.9"
                    fi

                    echo "🐍 Ensuring Python is Default..."
                    sudo update-alternatives --install /usr/bin/python python /usr/bin/$PYTHON_BIN 1

                    echo "🐍 Upgrading Pip..."
                    $PYTHON_BIN -m pip install --upgrade pip
                    
                    echo "📦 Installing Python Dependencies..."
                    $PYTHON_BIN -m pip install -r lambda-app/tests/requirements.txt
                    
                    echo "🔧 Installing pytest globally..."
                    $PYTHON_BIN -m pip install pytest  # Ensure pytest is installed

                    echo "☁️ Checking & Installing AWS SAM CLI..."
                    if ! command -v sam &> /dev/null
                    then
                        echo "🚀 Installing AWS SAM CLI..."
                        if ! curl -Lo aws-sam-cli.zip https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip --retry 5 --retry-delay 5; then
                            echo "❌ Curl failed, trying wget..."
                            wget -O aws-sam-cli.zip https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
                        fi
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
                    python3 -m pytest  # Use Python explicitly
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
