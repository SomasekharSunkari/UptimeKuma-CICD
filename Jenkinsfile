pipeline{
    agent any
    tools{
        jdk 'jdk-17'
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/SomasekharSunkari/Uptime-kuma2.0.git'            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=uptime \
                    -Dsonar.projectKey=uptime '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
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
                sh "trivy fs . > trivyfs_uptime.json"
            }
        }
        stage("Docker Build and  Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub-cred', toolName: 'docker'){
                       sh "docker build -t uptime ."
                       sh "docker tag uptime sunkarisomasekhar/uptime:latest "
                       sh "docker push sunkarisomasekhar/uptime:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sunkarisomasekhar/uptime:latest > trivy_uptime.json"
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop uptime | true"
                sh "docker rm uptime | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name uptime -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 sunkarisomasekhar/uptime:latest'
            }
        }
    }
    }
