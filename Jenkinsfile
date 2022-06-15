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

      stage('Docker Build and Push') {
            steps {
              sh 'printenv'
              sh 'docker build -t gvenkat/numericapp:"$GIT_COMMIT" .'
              sh 'docker push gvenkat/numericapp:"$GIT_COMMIT"'
            }
        } 
    }
}