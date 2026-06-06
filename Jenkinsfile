pipeline {
    agent any
    
    environment {
        AWS_CREDENTIALS_ID = 'aws-credentials' 
        TF_IN_AUTOMATION   = 'true'
    }
    
    tools {
        terraform 'terraform-default'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}")]) {
                    sh 'terraform init'
                }
            }
        }
        
        stage('Terraform Plan') {
            steps {
                withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}")]) {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }
        
        stage('Approval Gate') {
            steps {
                script {
                    // Pipeline pauses here and waits for an authorized user to click "Proceed"
                    input(
                        id: 'ApprovePlan', 
                        message: 'Review the Terraform Plan output above. Do you want to apply these changes?', 
                        ok: 'Apply Infrastructure'
                    )
                }
            }
        }
        
        stage('Terraform Apply') {
            steps {
                withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}")]) {
                    // Safely applies the exact plan generated in the previous stage
                    sh 'terraform apply -input=false tfplan'
                }
            }
        }
    }
    
    post {
        always {
            cleanWs() // Keeps the Jenkins workspace tidy
        }
        success {
            echo 'Infrastructure applied successfully!'
        }
        failure {
            echo 'Pipeline failed or was aborted during approval.'
        }
    }
}