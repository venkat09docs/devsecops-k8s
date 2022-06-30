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

      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      stage('Docker Build and Push') {
            steps {

              withDockerRegistry(credentialsId: 'docker-hub',url: '') {
                  sh 'printenv'
                  sh 'docker build -t gvenkat/numericapp:"$GIT_COMMIT" .'
                  sh 'docker push gvenkat/numericapp:"$GIT_COMMIT"'
              }              
            }
        } 

        stage('K8s Deployment - DEV') {
            steps {

              withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-config-file', namespace: '', serverUrl: '') {
                    sh "sed -i 's#replace#gvenkat/numericapp:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
              }                            
            }
        } 
    }
}