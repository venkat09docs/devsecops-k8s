pipeline {
  agent any

  stages {
      stage('Build Artifact Stage') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
    }
}