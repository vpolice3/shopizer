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
    	    mvn install -DskipTests=true
	    // mvn clean install
      	    
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
   
     stage('Push Docker image') {
	  steps{
		    withDockerRegistry([ credentialsId: "Docker_Hub", url: "" ]){
			
			bat "docker push debaduttapradhan1996/shopizer-app:latest"   
	  	   }
	   }
       } 
      stage('Deploy to AWS') {
            /*environment {
                DOCKER_HUB_LOGIN = credentials('docker-hub')
            }*/
            steps {
               /* withAWS(credentials: 'aws-credentials', region: env.REGION) {
                    bat '-PsubnetId=$SUBNET_ID -PdockerHubUsername=$DOCKER_HUB_LOGIN_USR'
                }*/
		sshagent(['Hackanthon-AWS-connection']) {
		   bat "ssh -o StrictHostKeyChecking=no ubuntu@18.209.225.172 sudo docker run debaduttapradhan1996/shopizer-app:latest"
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
