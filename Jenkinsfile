def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
    'UNSTABLE': 'danger'
]
pipeline {
    agent any
    environment {
        SCANNER_HOME=tool 'SonarScanner'
        SNYK_HOME   = tool name: 'Snyk'
    }
    tools {
        snyk 'Snyk'
    }
    stages {
        // Checkout To The Service Branch
        stage('Checkout To Mcroservice Branch'){
            steps{
                git branch: 'app-frontend-service', url: 'https://github.com/oluwa200/multi-microservices-application-project.git'
            }
        }
        // SonarQube SAST Code Analysis
        stage("SonarQube SAST Analysis"){
            steps{
                withSonarQubeEnv('Sonar-Server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=app-frontend-service \
                    -Dsonar.projectKey=app-frontend-service '''
                }
            }
        }
        // Providing Snyk Access
        stage('Authenticate & Authorize Snyk') {
            steps {
                withCredentials([string(credentialsId: 'Snyk-API-Token', variable: 'SNYK_TOKEN')]) {
                    sh "${SNYK_HOME}/snyk-linux auth $SNYK_TOKEN"
                }
            }
        }
        // Scan Service Dockerfile With Open Policy Agent (OPA)
        stage('OPA Dockerfile Vulnerability Scan') {
            steps {
                sh "docker run --rm -v ${WORKSPACE}:/project openpolicyagent/conftest test --policy docker-opa-security.rego Dockerfile || true"
            }
        }
        // Build and Tag Service Docker Image
        stage('Build & Tag Microservice Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Credential', toolName: 'docker') {
                        sh "docker build -t oluwachei/frontendservice:latest ."
                    }
                }
            }
        }
        // Execute SCA/Dependency Test on Service Docker Image
        stage('Snyk SCA Test | Dependencies') {
            steps {
                sh "${SNYK_HOME}/snyk-linux test --docker oluwachei/frontendservice:latest || true" 
            }
        }
        // Push Service Image to DockerHub
        stage('Push Microservice Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Credential', toolName: 'docker') {
                        sh "docker push oluwachei/frontendservice:latest "
                    }
                }
            }
        }
        // // Deploy to The Staging/Test Environment
        // stage('Deploy Microservice To The Stage/Test Env'){
        //     steps{
        //         script{
        //             withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'Kubernetes-Credential', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                sh 'kubectl apply -f deploy-envs/test-env/test-namespace.yaml'
        //                sh 'kubectl apply -f deploy-envs/test-env/deployment.yaml'
        //                sh 'kubectl apply -f deploy-envs/test-env/nodeport-service.yaml'  //NodePort Service
        //            }
        //         }
        //     }
        // }
        // // Perform DAST Test on Application
        // stage('ZAP Dynamic Testing | DAST') {
        //     steps {
        //         sshagent(['OWASP-Zap-Credential']) {
        //             sh 'ssh -o StrictHostKeyChecking=no ubuntu@34.71.9.167 "docker run -t zaproxy/zap-weekly zap-baseline.py -t http://172.31.6.49:30000/" || true'
        //                                                 //JENKINS_PUBLIC_IP                                                   //172.31.27.194:30000
        //         }
        //     }
        // }
        // // Production Deployment Approval
        // stage('Approve Prod Deployment') {
        //     steps {
        //             input('Do you want to proceed?')
        //     }
        // }
        // // // Deploy to The Production Environment
        // stage('Deploy Microservice To The Prod Env'){
        //     steps{
        //         script{
        //             withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'Kubernetes-Credential', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                sh 'kubectl apply -f deploy-envs/prod-env/prod-namespace.yaml'
        //                sh 'kubectl apply -f deploy-envs/prod-env/deployment.yaml'
        //                sh 'kubectl apply -f deploy-envs/prod-env/loadbalancer-service.yaml'  //LoadBalancer Service
        //             }
        //         }
        //     }
        // }
    }
    post {
    always {
        echo 'Slack Notifications.'
        slackSend channel: '#jean_moise--multi-microservices-alert', //update and provide your channel name
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job Name '${env.JOB_NAME}' build ${env.BUILD_NUMBER} \n Build Timestamp: ${env.BUILD_TIMESTAMP} \n Project Workspace: ${env.WORKSPACE} \n More info at: ${env.BUILD_URL}"
    }
  }
}
