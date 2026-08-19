pipeline {
 agent any 
 environment {
        awsRegion = 'us-east-1' 
        awsAccountId = "084129280874"  
        repoName = "springboot-demo"
        imageTag = "${env.BUILD_NUMBER}"
        ecrRegistry = "${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com"
        argocdAppName = "springboot-demo-app" // Replace with your actual ArgoCD Application name
        argocdServer = "argocd.yourdomain.com" // Replace with your ArgoCD server URL or use CLI context if pre-configured
  
 }

 stages{
   stage('Git-checkout'){
        steps {
           checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'Git_Login', url: 'https://github.com/intellectgit/java-maven-project-new.git']])
        }
   }
   stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
      stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
  stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.201.116.83:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }

//In software, a Code Smell is a warning from SonarQube that says: "Your application works right now, but the code is so messy, repetitive, or confusing that future changes will be slow, painful, and prone to breaking."

// bug 
// vulnerability 

 stage('Quality Gate') {
        steps {
            script {
                // 1. Run the quality gate without auto-aborting
                def qg = waitForQualityGate(abortPipeline: false, credentialsId: 'sonar-token')
                
                // 2. Check if the status is NOT OK (i.e., Failed, Error, or Warn depending on your settings)
                if (qg.status != 'OK') {
                    // Update the global build result so the post block or notifications recognize the failure
                    currentBuild.result = 'FAILURE'
                    
                    // 3. Send the failure report email specifically to Ramesh
                    def jobName = env.JOB_NAME
                    def buildNumber = env.BUILD_NUMBER
                    
                    def body = """
                        <html>
                        <body>
                        <div style="border: 4px solid red; padding: 10px;">
                        <h2>${jobName} - Build ${buildNumber}</h2>
                        <div style="background-color: red; padding: 10px;">
                        <h3 style="color: white;">Pipeline Failed: SonarQube Quality Gate Status was ${qg.status}</h3>
                        </div>
                        <p>The quality gate criteria were not met. Please check the <a href="${BUILD_URL}">SonarQube dashboard</a> and console output to fix the issues.</p>
                        </div>
                        </body>
                        </html>
                    """

                    emailext (
                        subject: "${jobName} - Build ${buildNumber} - Quality Gate FAILED",
                        body: body,
                        to: 'ramesh.tomcat@gmail.com',
                        from: 'jenkins@example.com',
                        replyTo: 'jenkins@example.com',
                        mimeType: 'text/html'
                    )
                    
                    // 4. Explicitly abort the pipeline
                    error("Pipeline aborted because the SonarQube Quality Gate failed (Status: ${qg.status}). Email notification sent to ramesh.tomcat@gmail.com.")
                } else {
                    echo "SonarQube Quality Gate passed successfully!"
                }
            }
        }
    }

//Common metrics evaluated by a Quality Gate include:
New Code Conditions (Clean as You Code): Modern SonarQube heavily focuses on newly introduced code. Common gates state that New Code must have:

Blocker or Critical Bugs and Vulnerabilities.Maintainability Rating of A (meaning zero critical code smells added).

Test Coverage Thresholds: Setting a minimum percentage of unit test coverage for new code (e.g., Coverage on New Code must be greater than 80%

Duplicated Code Limits: Ensuring that duplicated code blocks on new lines do not exceed a specific threshold (e.g., less than 3%).

Security Hotspots Reviewed: Ensuring that 100%of security hotspots identified in new code have been explicitly reviewed and cleared by a developer.
 
 stage('Build') {
            steps {
               sh "mvn package"
            }
        }
		
  stage('Docker-build'){
     steps{
      sh "docker build -t ${repoName}:${imageTag} ."
     }
  }
   stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html ${repoName}:${imageTag} "
            }
        }
        
stage('Push to AWS ECR') {
    steps {
        script {
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'ecr-login',                            accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
            // 1. Authenticate Docker with AWS ECR
            sh "aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${ecrRegistry}"

            // 2. Tag your built local image to match the ECR repository URI
            sh "docker tag ${repoName}:${imageTag} ${ecrRegistry}/${repoName}:${imageTag}"

            // 3. Push the image to AWS ECR
            sh "docker push ${ecrRegistry}/${repoName}:${imageTag}"
                        }
        }
    }
}
stage('Update Deployment File') {
    environment {
        GIT_REPO_NAME = "java-maven-project-new"
        GIT_USER_NAME = "intellectgit"    
    }
    steps {
        script {
          
            // 2. Bind the 'git-push' credential so we can safely use its password/token in the push URL
            withCredentials([usernamePassword(credentialsId: 'git-push', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                
                sh   'git config user.email "ramesh.tomcat@gmail.com"'
                sh   'git config user.name "ramesh ram"'
                sh  'sed -i "s/replaceImageTag/${imageTag}/g" guestbook/guestbook-ui-deployment.yaml'
                
                sh  'git add guestbook/guestbook-ui-deployment.yaml'
                sh  'git commit -m "Update deployment image to version ${imageTag}"'
                    
                    
                sh  'git push https://intellectgit:${GIT_TOKEN}@github.com/intellectgit/java-maven-project-new.git HEAD:master'
                
            }
        }
    }
}

   // --- NEW STAGE 1: ArgoCD Sync ---
   stage('ArgoCD Sync') {
       steps {
           script {
               // Assuming argocd CLI is installed on the Jenkins agent and logged in, 
               // or you can authenticate using credentials via token/username-password:
               // e.g., sh "argocd login ${argocdServer} --username admin --password \$ARGOCD_PASSWORD --insecure"
               
               echo "Triggering ArgoCD sync for application: ${argocdAppName}"
               sh "argocd app sync ${argocdAppName}"
           }
       }
   }

   // --- NEW STAGE 2: Deployment Validation ---
   stage('Deployment Validation') {
       steps {
           script {
               echo "Waiting for ArgoCD to complete sync and health check..."
               // This command waits until the application is healthy and synced (timeout after 300 seconds)
               sh "argocd app wait ${argocdAppName} --health --sync --timeout 300"
               
               // Optional: Add custom validation curl requests or health endpoint checks here
               echo "Deployment successfully verified and healthy!"
           }
       }
   }
   
}
 post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

           

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'jaiswaladi246@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
       }
    }
}
