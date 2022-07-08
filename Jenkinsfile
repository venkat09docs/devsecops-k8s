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

      stage('Sonar Scanner - SAST') {
          steps{
            withSonarQubeEnv('sonarqube') {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://52.74.218.166:9000 -Dsonar.login=c1ad7f5383ce943555f4018834e0a11f2b686829"
            }

            timeout(time: 2, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
              def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
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