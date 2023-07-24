pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }
        
      stage('Unit Tests and jacoco') {
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
             sh "mvn org.pitest:pitest-maven:scmMutationCoverage -Dinclude=ADDED,UNKNOWN -DmutationThreshold=50"
       }
     }
     stage('Docker Build and Push') {
           steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                  sh 'printenv'
                  sh 'sudo docker build -t drop149/devsecops-java-demo:""$GIT_COMMIT"" .'
                  sh 'docker push drop149/devsecops-java-demo:""$GIT_COMMIT""'
         
       }
     }

     }

     stage('K8S Deployment - DEV') {
           steps {
             parallel "Deployment": {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "bash k8s-deployment.sh"
             }
           }
	 
}

    }
  }

}

