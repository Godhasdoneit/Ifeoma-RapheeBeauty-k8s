pipeline {
    agent any
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS=credentials('716274dc-f41e-4e10-9f58-f501c9063a39')
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    }
    stages{
       stage('Clean workspace'){
           steps{
            cleanWs()
           }
       }
       stage('Git Checkout'){
            steps{    
               checkout scmGit(branches: [[name: '*/dev'], [name: '*/qa'], [name: '*/prod']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AfriTech-DevOps/Ifeoma-RapheeBeauty.git']])
            
            }
        }
        stage('Sonarqube Analysis'){
            steps{
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Rapheebeauty -Dsonar.projectName=Rapheebeauty"
                }
            }
        }
        }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Login to DockerHUB'){
            steps{
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                echo "login succeeded"
            }
        }
        stage('Trivy File Scan'){
            steps{
                script{
                    sh 'trivy fs . > trivy_result.txt'
                }
            }
        }
        stage('Docker Build'){
            steps{
                script{
                   def imageTag = determineTargetEnvironment() 
                   sh "docker build -t blesseddocker/ifeoma-rapheebeauty:${imageTag} ."
                   echo "Image Build Successfully"
                    
                }
            }
        }

        stage('Trivy Image Scan'){
            steps{
                script{
                    def imageTag = determineTargetEnvironment() 
                    sh "trivy image blesseddocker/ifeoma-rapheebeauty:${imageTag}"
                }
            }
        }
        stage('Docker push'){
            steps{
                script{
                    def imageTag = determineTargetEnvironment() 
                    sh "docker push blesseddocker/ifeoma-rapheebeauty:${imageTag}"
                    echo "Push Image to Registry"
                }
            }
        }
        
    }
}
def determineTargetEnvironment() {
    def branchName = env.BRANCH_NAME
    if (branchName == 'qa') {
        return 'qa'
    } else if (branchName == 'prod') {
        return 'prod'
    } else {
        return 'dev'
    }
}

