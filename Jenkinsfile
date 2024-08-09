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
                git branch: 'main', url: 'https://github.com/priya241302/DevSecOps-Project-Netflixapp.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
                    sh "docker build --build-arg TMDB_V3_API_KEY=df28cd0bb235035caab3f50793cba4d4 -t netflix ."
                    sh "docker tag netflix priya247/netflix:latest "
                    sh "docker push priya247/netflix:latest "
                    }
                }
            }
    }
        stage("TRIVY"){
            steps{
            sh "trivy image priya247/netflix:latest > trivyimage.txt"
        }
    }
        stage('Deploy to container'){
    steps{
        sh 'docker rm -f netflix '
        sh 'docker run -d --name netflix -p 8081:80 priya247/netflix:latest'
        
        }
   }

    }
}
