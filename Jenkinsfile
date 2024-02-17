pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean Workspace') {
            steps {
               cleanWs ()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch:'main',url:'https://github.com/srisainaidu/Youtube-clone-app.git'
            }
        } 
        stage('Sonarqube analysis') {
            steps {
               withSonarQubeEnv('sonar-server') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=youtube \
                   -Dsonar.projectKey=youtube'''
               }
            }
        }
        stage('quality gate') {
            steps {
               script {
                   waitForQualityGate abortpipeline: false, creditialsId:'sonar-token'
               }
            }
        }
        stage('Install Dependencies') {
            steps {
               sh "npm install"
            }
        }
        stage('Trivy Fs Scan') {
            steps {
               sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Build& Tag') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'Docker-cred', toolName: 'Docker') {
                       sh "docker build --build-arg REACT_APP_RAPID_API_KEY=56fb99185fmsha05f108c5b38ba2p1ca897jsn861c40b55861 -t youtube5 ."
                       sh "docker tag youtube srisainaidu/youtube5:latest "
                       sh "docker push srisainaidu/youtube5:latest "
                 }
               }
            }
        }
         stage("TRIVY"){
            steps{
                sh "trivy image srisainaidu/youtube5:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name youtube -p 3000:3000 srisainaidu/youtube5:latest'
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yml'
                    }
                }
            }
        }
        
    }
}
