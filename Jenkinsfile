pipeline{
    agent any
    tools{
        maven 'mvn3.9'
        jdk 'JDK17'
    }
    environment{
        registryCredential = 'ecr:us-east-1:awscreds_practice'
        imageName = '045712586082.dkr.ecr.us-east-1.amazonaws.com/vprofile-practice'
        vprofileRegistry = 'http://045712586082.dkr.ecr.us-east-1.amazonaws.com'
        cluster = 'vprofile-practice'
        service = 'vprofile-practice-svc'
    }
    stages{
        stage("Fetch the code"){
            steps{
                git branch:'docker', url:'https://github.com/m-s-sat/vprofile-project.git'
            }
        }
        stage('Build the code'){
            steps{
                sh 'mvn install -DskipTests'
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Test the code'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Code Analysis'){
            environment{
                scannerHome = tool 'sonar6.2'
            }
            steps{
                withSonarQubeEnv("sonarscanner"){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate'){
            steps{
                timeout(time:1, unit:'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Build Docker Image"){
            steps{
                script{
                    dockerImage = docker.build(imageName + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }
        stage("Upload App image"){
            steps{
                script{
                    docker.withRegistry(vprofileRegistry, registryCredential){
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage("Upload to ecs"){
            steps{
                script{
                    withAWS(credentials:'awscreds_practice',region:'us-east-1'){
                        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                    }
                }
            }
        }
    }
}