Managing Jenkins CI/CD pipeline credentials using AWS Secrets Manager is an industry-best-practice approach. Instead of storing passwords, API tokens, or keys directly inside Jenkins' internal database (or plain text in GitHub repositories), Jenkins fetches them dynamically from AWS at runtime.Part 1: How to Manage and Integrate ItThe cleanest and most standard way to link AWS Secrets Manager with Jenkins is using the AWS Secrets Manager Credentials Provider Plugin.  Step 1: Grant AWS Permissions to JenkinsYour Jenkins server/agent needs permission to read secrets. If Jenkins runs on an EC2 instance, ECS, or EKS, attach an IAM Role to it with the following permission policy:


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      
      "Action":  [    
         "secretsmanager:GetSecretValue",
        "secretsmanager:ListSecrets"
      ],
      "Resource": "*"
    }
  ]
}


Step 2: Store the Secret in AWS Secrets ManagerGo to the AWS Console, open Secrets Manager, and create a secret (e.g., named prod/docker/credentials).To make it automatically discoverable by Jenkins, add specific AWS Tags to the secret (such as jenkins:credentials:type set to string or usernamePassword). Alternatively, the plugin can map secrets by matching their exact name. 


Step 3: Use the Secret in your Jenkinsfile PipelineOnce the plugin is installed and configured, you don’t need to manually create credentials in the Jenkins UI anymore. You can reference your AWS secret directly inside your Declarative Pipeline using the credentials()   


pipeline {
    agent any

    environment {
        // Automatically fetches the secret value from AWS Secrets Manager at runtime
        DOCKER_AUTH = credentials('prod/docker/credentials')
    }

    stages {
        stage('Deploy to ECR') {
            steps {
                script {
                    // DOCKER_AUTH_USR and DOCKER_AUTH_PSW are exposed safely to the block
                    echo "Deploying using credentials fetched live from AWS Secrets Manager"
                }
            }
        }
    }
}


For more details refer document & video 

document - https://plugins.jenkins.io/aws-secrets-manager-credentials-provider/

Video - https://www.youtube.com/watch?v=lp1ZdJIUkQk&t=1130s
