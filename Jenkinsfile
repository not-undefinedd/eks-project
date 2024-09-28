pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID= "844085826365"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="my-image-repo"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "844085826365.dkr.ecr.us-east-2.amazonaws.com/my-image-repo"
    }
    stages {
        
        stage('Build with maven') {
            steps {
                sh 'cd SampleWebApp && mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        
        }

        stage('Code Quality Scan') {

           steps {
                  withSonarQubeEnv('sonar-server') {
                    sh "mvn -f SampleWebApp/pom.xml sonar:sonar"
               }
            }
       }

        stage('Quality Gate') {
          steps {
                 waitForQualityGate abortPipeline: true
              }
        }     
        
        
         stage('Logging into AWS ECR') {
                     environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')

                   }
                     steps {
                       script{

                         sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }

            }
        }
      
         stage('Building image') {
            steps{
              script {
                dockerImage = docker build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
               }
            }
        }
        
        stage('Pushing to ECR') {
          steps{
            script {
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                sh """docker push ${REPOSITORY_URI}:$IMAGE_TAG"""
              }
            }
         }
         
         stage('pull image & Deploying UI application on eks cluster DEV') {
                    environment {
                       AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                       AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                 }
                    steps {
                      script{
                        dir('kubernetes/') {
                          sh 'aws eks update-kubeconfig --name myAppp-eks-cluster --region us-east-2'
                          sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                          sh 'helm upgrade --install --set image.repository="$REPOSITORY_URI" --set image.tag="${IMAGE_TAG}" myjavaapp myapp/ '

 
                        }
                    }
               }
            }
    }
}
