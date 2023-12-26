pipeline{
    agent any
    tools{
        jdk 'jdk17'
        // nodejs 'node16'
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        // GIT_REPO_NAME = "Tetris-manifest"
        // GIT_USER_NAME = "rameshkumarvermagithub"      # change your Github Username here
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/RAMESHKUMARVERMAGITHUB/spring-cloud-kubernetes.git'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'cd kubernetes-configmap-reload && mvn clean package'
            }
        }
        stage("Test Application"){
            steps {
                sh "cd kubernetes-configmap-reload && mvn test"
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar') {
                        sh "cd kubernetes-configmap-reload && mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage("Sonarqube Analysis "){
        //     steps{
        //         withSonarQubeEnv('sonar-server') {
        //             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=complete-prodcution-e2e-pipeline \
        //             -Dsonar.projectKey=complete-prodcution-e2e-pipeline '''
        //         }
        //     }
        // }
        // stage("quality gate"){
        //   steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar' 
        //         }
        //     } 
        // }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "cd kubernetes-configmap-reload && trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){ 
                       // sh "docker pull chydinma/app:1"
                      sh "cd kubernetes-configmap-reload && docker build -t rameshkumarverma/kubernetes-configmap-reload:latest ."
                       // sh "docker tag chydinma/app:1 rameshkumarverma/myprojectapp:latest"
                       sh "cd kubernetes-configmap-reload && docker push rameshkumarverma/kubernetes-configmap-reload:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "cd kubernetes-configmap-reload && trivy image rameshkumarverma/kubernetes-configmap-reload:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to Kubernets'){
            steps{
                script{
                    dir('kubernetes-configmap-reload') {
                      withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      sh 'kubectl delete --all pods'
                      sh 'kubectl apply -f deployment.yml'
                      sh 'kubectl apply -f service.yml'
                      }   
                    }
                }
            }
        }
    }
}
