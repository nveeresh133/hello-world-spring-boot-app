pipeline {
 agent any
	parameters {
        choice (name: 'Service', description: 'Service to build', 
            choices: [
            'frontend-service',
            'backend-service'
            ])
//         booleanParam (defaultValue: false, description: 'Update ECS Service', name: 'Promote CD Deployment')
        choice (name: 'deploymentconfig', description: 'Selecting the deployment configuration type', choices: [
                'CodeDeployDefault.ECSAllAtOnce',
                'CodeDeployDefault.ECSLinear10PercentEvery1Minutes',
                'CodeDeployDefault.ECSLinear10PercentEvery3Minutes',
		'CodeDeployDefault.ECSCanary10Percent5Minutes',
		'CodeDeployDefault.ECSCanary10Percent15Minutes'])
        }
 environment {
 AWS_ACCOUNT_ID="823226410025"
 AWS_DEFAULT_REGION="us-east-2" 
 IMAGE_REPO_NAME="spring-private"
 IMAGE_TAG="${BUILD_NUMBER}"
 REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
 S3_BUCKET = "my-tf-test-bucket12344"
 S3_OBJECT_KEY = "myapp-${env.BUILD_NUMBER}.zip"
 CODEDEPLOY_APPLICATION = "dev-code-deploy"
 CODEDEPLOY_DEPLOYMENT_GROUP = "dev-code-deploy-group"
//  deploymentConfigName = "${params.deployment-config}"
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
				sh "zip -r myapp-${env.BUILD_NUMBER}.zip appspec.yaml"
			}
		}
stage('Upload to S3') {
            steps {
                sh "aws s3 cp myapp-${env.BUILD_NUMBER}.zip s3://${S3_BUCKET}/${S3_OBJECT_KEY}"
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
 sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:latest"
 sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
 sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest"

 }
 }
 }
 stage('Deploy to CodeDeploy') {
 steps {
 script { 
 sh "echo ${params.deploymentconfig}"
//  sh "echo ${deploymentConfigName}"
 sh "aws deploy update-deployment-group --application-name ${CODEDEPLOY_APPLICATION} --current-deployment-group-name ${CODEDEPLOY_DEPLOYMENT_GROUP} --deployment-config-name ${params.deploymentconfig} --region ${AWS_DEFAULT_REGION}"
 sh "aws deploy create-deployment --application-name ${CODEDEPLOY_APPLICATION} --deployment-group-name ${CODEDEPLOY_DEPLOYMENT_GROUP} --s3-location bucket=${S3_BUCKET},key=${S3_OBJECT_KEY},bundleType=zip --region ${AWS_DEFAULT_REGION}"
		}
            }
        }
 
}
}

