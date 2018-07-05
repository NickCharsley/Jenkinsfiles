#!groovy

pipeline {
    
    agent any
    options { 
        skipStagesAfterUnstable() 
    }
    environment { 
        branch = '${PROJECT_NAME}-${VERSION_NUMBER}.${BUILD_NUMBER}'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.           
        mvnHome = tool 'M3'
    }
    stages {
        stage('Checkout') {
            steps {
                // Create Branch...
                sshagent (credentials: ['77bcb061-7eae-4009-a6cf-2b9c9c5331df']) {
                    sh "git checkout -b ${branch}"
                }
                //Update the POM version
                sh "'${mvnHome}/bin/mvn' versions:set -DnewVersion=${VERSION_NUMBER}.${BUILD_NUMBER}"
            }
        }
        stage('Build') {
            steps {
                sh "'${mvnHome}/bin/mvn' clean package -Dmaven.test.skip=true"
            }
        }
       stage('Test') {
           steps {
                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore test -Dtest=unitTests,requirementsTest"
           }
       }
       stage('Results') {
            steps {
                junit '**/target/surefire-reports/TEST-*.xml'
                archive 'target/*.jar'
            }
       }
    }
    post {
        success {
            steps {
                sshagent (credentials: ['77bcb061-7eae-4009-a6cf-2b9c9c5331df']) {
                    sh "git commit -a -m 'New Release Candidate ${branch}'"
                    sh "git push origin ${branch}"
                }      
            }
        }
    }
}
