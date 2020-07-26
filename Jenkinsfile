pipeline {
 agent any
  stages {
    stage('SCM') {
      steps {
         git 'https://github.com/Debadutta-Pradhan/shopizer.git'
       }
    }
   stage("Build") {
     steps {
	   
    bat ''' 
    	    mvn clean install -DskipTests=true
	    mvn clean install
      	    
       '''
   }
   }
	  


   stage('Approval') {
            // no agent, so executors are not used up when waiting for approvals
            agent none
            steps {
                script {
                    def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                    sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	
    stage('Build images') {
	      steps {
		bat '''
			  cd sm-shop
			  docker build -f "Dockerfile" -t debaduttapradhan1996/shopizer-app:latest .
		  
		'''
	      }
       }
   
      stage('Push image') {
         steps {
           withDockerRegistry([url: "https://us-east-1.console.aws.amazon.com/ecr/repositories/sophizer-app/?region=us-east-1",credentialsId: "ecr:us-east-1:shopizer_id"]) {
           bat 'docker push debaduttapradhan1996/shopizer-app:latest'
               }
        }
  }
}
post {
        always {
            //archiveArtifacts artifacts: 'generatedFile.log', onlyIfSuccessful: true
          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}
