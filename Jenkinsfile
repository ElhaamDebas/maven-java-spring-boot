pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-secret-access-key')
        AWS_S3_BUCKET = "maven-teamcity"
        ARTIFACT_NAME = "sample-test-2.jar"
        AWS_EB_APP_NAME = "dotnet-web-server"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Dotnetwebserver-env"
        
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
            post{
                always{
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        //stage('Quality Scan'){
        //    steps {
        //       sh '''
        //            mvn clean verify sonar:sonar \
        //            -Dsonar.projectKey=java-mshaikh \
        //            -Dsonar.host.url=http://$SONAR_IP \
        //            -Dsonar.login=$SONAR_TOKEN
        //        '''
        //    }
        //}
        stage('Package') {
            steps {
                sh "mvn package"
            }
            post{
                success{
                    archiveArtifacts artifacts: 'target/sample-test-2.jar', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
            steps {
                sh "aws configure set region us-east-1"
                sh "aws s3 cp ./target/sample-test-2.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
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
