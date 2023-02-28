pipeline{
    agent any
    environment {
        PATH = "$PATH:/opt/maven/bin"
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.89.62.95:8081"
        NEXUS_REPOSITORY = "maven-central-repository"
        NEXUS_CREDENTIAL_ID = "NEXUS_CRED"
    }
    stages{
       stage('GetCode'){
            steps{
                git 'https://github.com/dapiroy/hello-world.git'
            }
       }
     
       stage('Build'){
            steps{
                sh 'mvn clean package'
            }
       }
       stage('UNIT testing'){
            steps{
                script{
                    sh 'mvn test'
                }
            }
       }
       stage('SonarQube analysis') {
//    def scannerHome = tool 'SonarScanner 4.7.0';
             steps{
             withSonarQubeEnv('sonarqube-9.9') { 
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn sonar:sonar"
             }
             }
       }
    //    stage("Build Docker Image") {
    //         steps{
    //             script{
    //                 dockerImage = docker.build imageName
    //             }
    //         }
    //     }
        
       stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    
                    nexusArtifactUploader artifacts: [[artifactId: 'pom.maven-project', classifier: '', file: 'webapp/target/webapp.war', type: 'war']],
                    credentialsId: 'NEXUS_CRED', 
                    groupId: 'pom.com.example.maven-project', 
                    nexusUrl: '3.89.62.95:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'maven-central-repository', 
                    version: 'pom.1.0-SNAPSHOT'
                }
            }
        }
        
        stage("Create an EKS Cluster") {
            steps {
                script {
                    dir('aws_infrastructure') {
                        sh "terraform init"
                        sh "terraform apply -auto-approve"
                    }
                }
            }
       }
        
        stage("Deploy to EKS") {
            steps {
                script {
                    dir('k8s') {
                        sh "aws eks update-kubeconfig --name my-eks"
                        sh "kubectl apply -f regapp-deploy.yml"
                        sh "kubectl apply -f regapp-service.yml"
                    }
                }
            }
        }
    }
    
}