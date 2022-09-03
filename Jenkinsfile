pipeline {
  agent any
  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }
    stage('Unit Tests - JUnit and Jacoco') {
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

      stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
        sh 'printenv'
        sh 'docker build -t vieenodp/numeric-app:""$GIT_COMMIT"" .'
        sh 'docker push vieenodp/numeric-app:""$GIT_COMMIT""'
      }
      }
    }

    stage('Kubernetes Deployment - Dev') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
        sh 'printenv'
        sh "sed -i 's#replace#vieenodp/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
        sh "kubectl apply -f k8s_deployment_service.yaml"
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
  }
}