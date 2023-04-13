pipeline {
    agent any
    parameters {
        choice(
            choices: ['plan', 'apply', 'show', 'destroy'],
            description: 'Terraform action to apply',
            name: 'action')
        choice(
            choices: ['dev', 'qa'],
            description: 'deployment environment',
            name: 'ENVIRONMENT')
        string(defaultValue: "ap", description: 'aws region', name: 'AWS_REGION')
    }
    stages {
        stage('init') {
            steps {
                sh 'rm -rf .terraform'
                sh 'terraform init -no-color -backend-config="${ENVIRONMENT}/${ENVIRONMENT}.tfbackend" '           
            }
        }
        stage('validate') {
            steps {
                sh 'terraform validate -no-color'
            }
        }
        stage('plan') {
            when {
                expression { params.action == 'plan' || params.action == 'apply' }
            }
            steps {
                sh 'terraform plan -no-color -input=false -out=tfplan -var "aws_region=${AWS_REGION}" --var-file=${ENVIRONMENT}/${ENVIRONMENT}.tfvars'
            }
        }
        stage('approval') {
            when {
                expression { params.action == 'apply'}
            }
            steps {
                sh 'terraform show -no-color tfplan > tfplan.txt'
                script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }
        stage('apply') {
            when {
                expression { params.action == 'apply' }
            }
            steps {
                sh 'terraform apply -no-color -input=false tfplan'
            }
        }
        stage('destroy') {
            when {
                expression { params.action == 'destroy' }
            }
            steps {
                script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Delete the stack?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
                sh 'terraform destroy -no-color -force --var-file=${ENVIRONMENT}/${ENVIRONMENT}.tfvars'
            }
        }
    }
}