pipeline {
 agent any

 environment {

     AWS_ACCESS_KEY_ID     = credentials('faisal-aws-secret-key-id')
     AWS_SECRET_ACCESS_KEY = credentials('faisal-aws-secret-access-key')

     AWS_S3_BUCKET         = "faisal-b-alotibi-belt2d2-artifacts-123456"
     AWS_REGION            = "me-south-1"
     ARTIFACT_NAME         = "spring-boot-rest-services-0.0.1-SNAPSHOT.jar"
     AWS_EB_APP_NAME       = "FAISAL-ALOTIBI-eb-mv"
     AWS_EB_APP_VERSION    = "${BUILD_ID}"
     AWS_EB_ENVIRONMENT    = "Faisalalotibiebmv-env"

     SONAR_PROJECT_KEY     = "onsite-Faisal-Alotibi-B2D2"
     SONAR_IP              = "52.23.193.18"
     SONAR_TOKEN           = "sqp_047cf336f00d212719a303d30773363b336e57e2"

 }

 stages {
     stage('Validate') {
         steps {                
             sh "mvn validate"
             sh "mvn clean"
         }
     }

      stage('Build') {
         steps {                
             sh "mvn compile"
         }
     }

     stage('Test') {
         steps {                
             sh "mvn test"
         }

         post {
             always {
                 junit '**/target/surefire-reports/TEST-*.xml'
             }
         }
     }

     stage('Quality Scan'){
         steps {
             sh '''
             mvn clean verify sonar:sonar \
                 -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                 -Dsonar.host.url=http://$SONAR_IP \
                 -Dsonar.login=$SONAR_TOKEN
             '''
         }
     }

     stage('Package') {
         steps {                
             sh "mvn package"
         }

         post {
             success {
                 archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false                   
             }
         }
     }

     stage('Publish artefacts to S3 Bucket') {
         steps {
             sh "aws configure set region $AWS_REGION"
             sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"                
         }
     }

     stage('Deploy') {
         steps {
             sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
             sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
         }
     }        
 }
}
