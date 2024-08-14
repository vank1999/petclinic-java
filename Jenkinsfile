pipeline {
    agent any

    tools {
        jdk 'jdk11'
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

        stage('sonarqube-analysis'){
            steps{
              withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Petclinic '''
                }
            }
        }

    }
}