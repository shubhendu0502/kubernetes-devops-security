pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
      stage('Unit Test- Jacoco, Junit') {
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
        stage('Sonarqube -SAST') {
            steps {
              sh " mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://65.1.13.101:9000 -Dsonar.login=sqp_485795b2c2a543d52ea4530f4b64b727fa8fe4ac"
            }
        }
        stage('Docker build and push') {
            steps {
              withDockerRegistry([credentialsId:"dockerhub", url:""]){
                sh 'printenv'
                sh 'docker build -t shubhendu0502/numeric-app:""$GIT_COMMIT"" . '
                sh 'docker push shubhendu0502/numeric-app:""$GIT_COMMIT""'
              }
            }
        }
        stage('Kubernetes Deployment -DEV') {
            steps {
              withKubeConfig([credentialsId:'kubeconfig']){
                sh "sed -i 's#replace#shubhendu0502/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
        }
    }
}
