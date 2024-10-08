pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages{
        stage('Git-checkout'){
            steps{
                git branch: 'main', credentialsId: 'git-cred',
                url: 'https://github.com/vank1999/petclinic-java.git'
            }
        }

        stage('code-compile'){
            steps{
              sh "mvn clean compile"
            }
        }

        stage('unit-tests'){
            steps{
              sh "mvn test"
            }
        }

        stage('Trivy scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('sonarqube-analysis'){
            steps{
              withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Petclinic '''
                }
            }
        }

        stage('OWASP scan') {
            steps {
               dependencyCheck additionalArguments: '', odcInstallation: 'DP-check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Code-Build') {
            steps {
               sh "mvn clean install"
            }
        }

        stage('Docker Build') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t  vank1999/petclinic:latest . "
                 }
            }
        }
      }

      stage('Trivy Scan Image') {
        steps {
        sh "trivy image --format table -o trivy-image-report.html vank1999/petclinic:latest"
        }
      }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker push  vank1999/petclinic:latest "
                 }
            }
        }
      }

      stage('Deploy using docker') {
            steps {
               sh "docker run -d --name petclinic -p 8082:8082 vank1999/petclinic:latest"
            }
        }

    stage('Deploy to Tomcat') {
            steps {
               sh " sudo cp /var/lib/jenkins/workspace/petclinic/target/petclinic.war /opt/tomcat/webapps/ "
            }
        }

    }
}