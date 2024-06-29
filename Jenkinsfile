pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Code CheckOut') {
            steps {
                git branch: 'main', url: 'https://github.com/AWS-DevOps-Learn/Ekart.git'
            }
        }
        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Maven Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=true '
            }
        }        
        stage('SonarQubeTests') {
            steps {
             // withSonarQubeEnv(credentialsId: 'sonar-token') {
             // server configured globally in system
                withSonarQubeEnv('sonar') { 
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
               }
            }
        } 
        stage('OWASP Dependency Check') {
            steps {
                //scan pom.xml
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC' 
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Package') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Upload To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                }
            }
        }  
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                    sh "docker build -t dockerraj24/ekart:latest -f docker/Dockerfile ."
                  }
               }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image dockerraj24/ekart:latest > trivy-report.txt"
            }
        }
        stage('Docker Push & Tag Image') {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                    sh "docker push dockerraj24/ekart:latest"
                  }
                }
            }
        }
        stage('kubernetes Deploy') {
            steps {
                script {
                 // withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.38.46:6443') {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://172.31.38.46:6443/']]) {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                  }
                }
            }
        }
    }//satges closing
}//pipeline closing
