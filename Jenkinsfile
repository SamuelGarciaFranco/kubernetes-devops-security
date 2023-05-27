pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        } 

      stage('Unit Test - Junit and Jacoco') {
            steps {
              sh "mvn test"
            }
        /*    post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            } */
        }  

      stage('Mutation Test - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
      }

      stage('SonarQube - SAST (Autenticacion SonarQube)') {
              steps {
                  withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar \
                        -Dsonar.projectKey=numeric-application \
                        -Dsonar.host.url=http://devsecops-demo-pro.eastus.cloudapp.azure.com:9000" 
                  }
                  timeout(time: 2, unit: 'MINUTES') {
                    script {
                      waitForQualityGate abortPipeline: true
                }
              }
            }
        } 
/* -Dsonar.login=sqp_aa34e578253e91b3ee597b56156ed770fbfdb538" */

      stage('Vulnerability Scan - Docker') {
            steps {
                sh 'mvn dependency-check:check'
            }
          }

      stage('Docker Build and push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'docker build -t nanobyte7/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push nanobyte7/numeric-app:""$GIT_COMMIT""'
              }
            }
        }  
  
      stage('Kubernetes Deployment - DEV') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#nanobyte7/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
        }     
    }
  post { 
        always { 
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      /*  success {

        }
        failure {

        } */
    }
}
