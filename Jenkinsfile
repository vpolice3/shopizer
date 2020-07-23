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
   stage('SonarQube') {
                steps {
                   bat '''
                     
              mvn sonar:sonar \
                    -Dsonar.projectKey=shopizer \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=b671d65077153b089e410f7281ce474c4d27fd4d  
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
	  
       stage('Publish') {
      when {
        branch '2.12.0'
      }
      steps {
	     withRegistry('https://registry.hub.docker.com/', 'docker_Hub'){
		bat '''
		
		 docker push debaduttapradhan1996/shopizer-app:latest
		 
		 '''
		  
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