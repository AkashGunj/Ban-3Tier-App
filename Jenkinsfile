pipeline {
    agent any
    
    tools{
        maven 'maven3'
        
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        //Checkout Code from GitHub 
        stage('Code Checkout') {
            steps {
              checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-cred', url: 'https://github.com/AkashGunj/Ban-3Tier-App.git']])
            }
        }
        
        //Compile source code using maven
        stage('Code Compile') {
            steps {
              sh "mvn compile"
            }
        }
        
        //Execute Unit test cases
        stage('Unit Test') {
            steps {
              sh "mvn test"
            }
        }
        
        //scan filesystem with Trivy
        stage('Filesystem Scan - Trivy') {
            steps {
                script{
                    def result = sh (
                    script: "trivy fs --format table -o fs-report.html --severity HIGH,CRITICAL --exit-code 0 .",
                    returnStatus: true
                    )
                
                
                if(result !=0){
                  error "High or Critical vulnerabilities found in the filesystem. Failing the build."  
                }
            }
                
                
              //sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        //Execute Sonarqube
       stage('Sonarqube Execution') {
            steps {
                withSonarQubeEnv('sonar') {
                   sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bankapp-new -Dsonar.projectKey=multi-tier-new -Dsonar.java.binaries=target" 
                }
            }
        }
        
        stage('Package-App') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Docker Image build') {
            steps {
                script{
                        // This step should not normally be used in your script. Consult the inline help for details.
                        withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                        // some block
                        sh "docker image build -t akashgun/bank-app:latest ."
                    }
                }
            }
        }
        
        stage('Docker Image publish') {
            steps {
                script{
                        // This step should not normally be used in your script. Consult the inline help for details.
                        withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                        // some block
                        sh "docker image push akashgun/bank-app:latest"
                    }
                }
            }
        }
        
        stage('Deploy Test') {
            steps {
                script{
                        // This step should not normally be used in your script. Consult the inline help for details.
                    sh "docker-compose -f /var/jenkins_home/workspace/Bank-3Tier-Application/docker-compose.yml down -d"    
                     sh "docker-compose -f /var/jenkins_home/workspace/Bank-3Tier-Application/docker-compose.yml up -d"
                }
            }
        }
             
    }
}
