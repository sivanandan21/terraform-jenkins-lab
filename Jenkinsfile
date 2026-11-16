pipeline {
    agent { label 'terraform-agent' }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        TF_IN_AUTOMATION = 'true'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Terraform Version') {
            steps { sh 'terraform version' }
        }

        stage('Terraform Format') {
            steps { sh 'terraform fmt -check -recursive' }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']]) {
                    sh 'terraform init -input=false -migrate-state -force-copy'
                }
            }
        }

        stage('Terraform Validate') {
            steps { sh 'terraform validate' }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']]) {
                    sh 'terraform plan -input=false -out=tfplan'
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Approve AWS infrastructure deployment?',
                          ok: 'Deploy'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']]) {
                    sh 'terraform apply -input=false tfplan'
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f tfplan'
        }
    }
}
