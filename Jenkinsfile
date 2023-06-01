pipeline {
 agent any
 environment {
 AWS_ACCOUNT_ID="823226410025"
 AWS_DEFAULT_REGION="us-east-2" 
 IMAGE_REPO_NAME="spring-private"
 IMAGE_TAG="latest"
 REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
 }
 
 stages {
 
 stage('Logging into AWS ECR') {
 steps {
 script {
 sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
//   aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 823226410025.dkr.ecr.us-east-2.amazonaws.com
 }
 
 }
 }
 
 stage('Clone repository') { 
            steps { 
                script{
               // git credentialsId: 'github-auth', url: 'https://github.com/nveeresh133/springboot-deployment.git'
                git credentialsId: 'github-auth', url: 'https://github.com/nveeresh133/hello-world-spring-boot-app.git'
                }
            }
        }
  
 stage('Build'){
  			steps {
		   		sh '''mvn clean package'''
			}
		}

// 		stage('Test'){
// 			steps{
// 				sh '''mvn test'''
// 			}
// 		}
 
 // Building Docker images
 stage('Building image') {
 steps{
 script {
 dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
 }
 }
 }
 
 // Uploading Docker images into AWS ECR
 stage('Pushing to ECR') {
 steps{ 
 script {
 sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
 sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
 }
 }
 }
 stage('Deploy to CodeDeploy') {
            steps {
                script {
                    withAWS( region: 'us-east-2') {
                        // Create a new AWS CodeDeploy deployment
                        def deployment = awsDeployCreateDeployment(applicationName: 'dev-code-deploy', deploymentGroupName: 'dev-code-deploy-group')

 

                        // Upload your application revision to S3
                        def s3ObjectKey = "dev-app-${env.BUILD_NUMBER}.zip"
                        def s3Bucket = 'my-tf-test-bucket12344'
		        def bundleType = 'zip'
                        awsS3Upload(file: 'dev-app-${env.BUILD_NUMBER}.zip', bucket: s3Bucket, path: s3ObjectKey)

 

                        // Register the uploaded revision with AWS CodeDeploy
                        awsDeployRegisterRevision(applicationName: 'dev-code-deploy', deploymentGroupId: deployment.deploymentGroupId)

 

                        // Deploy the registered revision with AppSpec file
                        awsDeployDeploy(deploymentId: deployment.deploymentId, fileExistsBehavior: 'OVERWRITE', appSpecFile: 'appspec.yaml')
                    }
                }
            } 
 }
}
}
