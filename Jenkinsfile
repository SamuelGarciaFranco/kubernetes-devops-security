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
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }  
      stage('Mutation Test - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
              always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
            }
        }

<<<<<<< HEAD

      stage('SonarQube - SAST') {
            steps {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo-pro.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_aa34e578253e91b3ee597b56156ed770fbfdb538"
            }
        } 

=======
>>>>>>> 0cf6a2209a513c56f3a698ea3a738cde85ea1e1b
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
}
