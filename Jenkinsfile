pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/kokareamol/Devsecops-Reddit-Clone.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit \
                    -Dsonar.projectKey=Reddit '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-cred' 
                }
            } 
        }
        
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t reddit ."
                       sh "docker tag reddit kokareamol/reddit:latest "
                       sh "docker push kokareamol/reddit:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image kokareamol/reddit:latest > trivy.txt" 
            }
        }
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name reddit -p 3000:3000 kokareamol/reddit:latest'
            }
        }



        
        
    }
}

