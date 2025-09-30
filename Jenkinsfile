pipeline {
    agent any
    
    tools{
        jdk "jdk"
        maven "maven3"
    }
    
    environment{
        SCANNER_HOME=tool "sonar-scanner"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Hitesh-Bhalotia/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Trivy File system scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                    dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=EKART '''
                }   
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings-xml'){
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId:'docker-cred', toolName:'docker'){
                        sh "docker build -t shopping-cart:dev -f docker/Dockerfile ."
                        sh "docker tag shopping-cart:dev hiteshbhalotia/shopping-cart:dev"
                    }
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    withDockerRegistry(credentialsId:'docker-cred', toolName:'docker'){
                        sh "docker run -d --name ekart -p 8070:8070 hiteshbhalotia/shopping-cart:dev"
                    }
                }
            }
        }
    }
}
