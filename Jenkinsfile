pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Vaishnavin0303/Zomato-Clone', credentialsId: 'git-credentials'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
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
                dependencyCheck additionalArguments: '--scan ./ --out dependency-check-report', odcInstallation: 'DP-Check'
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
                   withDockerRegistry(credentialsId: 'docker'){   
                       sh "docker build -t zomato ."
                       sh "docker tag zomato vaishnavi731/zomato:latest "
                       sh "docker push vaishnavi731/zomato:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image vaishnavi731/zomato:latest > trivy.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh '''
                    docker pull vaishnavi731/zomato:latest
                    docker rm -f zomato || true
                    docker run -d --name zomato -p 3000:3000 vaishnavi731/zomato:latest
                '''
          }
        }
    }
}
