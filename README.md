# AWS Lambda Deployment with Jenkins CI/CD Pipeline

## Overview
This project automates the deployment of an AWS Lambda function using a Jenkins CI/CD pipeline. The pipeline follows a structured approach to **setup**, **test**, **build**, and **deploy** the Lambda function using **AWS SAM (Serverless Application Model).**


## Prerequisites
Before setting up the Jenkins pipeline, ensure you have:
- **AWS CLI** installed and configured with credentials.
- **AWS SAM CLI** installed.
- **Jenkins** installed with the following plugins:
  - Pipeline
  - AWS Credentials
- **IAM Role & Policies** configured to allow Jenkins to deploy Lambda functions.
- **Jenkins environment variables** set for AWS credentials.

## Jenkins Pipeline Stages
The pipeline consists of four main stages:

### 1. **Setup**
- Installs necessary system dependencies.
- Installs Python and required dependencies.
- Ensures AWS SAM CLI is installed.

### 2. **Test**
- Runs unit tests using `pytest` to validate Lambda function behavior.

### 3. **Build**
- Uses AWS SAM to build the Lambda function package.

### 4. **Deploy**
- Deploys the Lambda function to AWS using AWS SAM.

## Obtaining API Gateway URL After Deployment
After each successful deployment, you can retrieve the **API Gateway endpoint** by running:

```sh
aws cloudformation describe-stacks --stack-name lambda-app --query "Stacks[0].Outputs"

## Jenkinsfile
Below is the Jenkins pipeline script used in this project:

```groovy
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
                    sudo apt install -y python3-pip unzip curl lsb-release

                    echo "🐍 Installing Python dependencies..."
                    python3 -m pip install --upgrade pip
                    python3 -m pip install -r lambda-app/hello_world/requirements.txt

                    echo "☁️ Checking & Installing AWS SAM CLI..."
                    if ! command -v sam &> /dev/null
                    then
                        echo "🚀 Installing AWS SAM CLI..."
                        curl -Lo aws-sam-cli.zip https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
                        unzip -o aws-sam-cli.zip -d sam-installation
                        sudo ./sam-installation/install --update
                        rm -rf aws-sam-cli.zip sam-installation
                    else
                        echo "✅ AWS SAM CLI is already installed."
                    fi
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    echo "🧪 Running Tests..."
                    python3 -m pytest lambda-app/tests/
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


