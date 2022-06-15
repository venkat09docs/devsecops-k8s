pipeline {
  agent any

  stages {
      stage('Build Artifact Stage') {
            steps {
              sh "mvn clean package -DskipTests=true"
              // archive 'target/*.jar' 
              archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
            }
        } 

      stage('Unit Tests - Junit') {
            steps {
              sh "mvn test"                            
            }
            post{
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
      }  
    }
}