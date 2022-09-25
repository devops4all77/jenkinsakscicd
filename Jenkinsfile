pipeline{
    agent any
    environment {
      DOCKER_TAG = getVersion()
	  aksrg = "sprinrg"
	  akscls = "springaks"
	  acrname = "acrjen1122443.azurecr.io"
	  acr_username = "acrjen1122443"
	  AZURETENANTID = "0c508b0d-c4b2-4a38-93c7-5947fb5a5656"
	  AZURESUBSCRIPTIONID = "e44c9379-2fc4-4ec0-8945-b7fd3ff798bd"
    }
    stages{
        stage('init'){
            steps{
                checkout scm
            }
        }
        
        stage('Build'){
            steps{
                sh "mvn clean package"
				
                sh "sudo docker build . -t ${acrname}/helloapp:latest"
               withCredentials([usernamePassword(credentialsId: 'acrauth', passwordVariable: 'acrpwd', usernameVariable: 'acruser')]) {
                    sh "sudo docker login ${acrname} -u ${acruser} -p ${acrpwd}"
                }
              
                sh "sudo docker push ${acrname}/helloapp:latest "
            }
        }
        
        stage('Deploy'){
            steps{
			withCredentials([usernamePassword(credentialsId: 'spauth', passwordVariable: 'AZURECLIENTSECRET', usernameVariable: 'AZURECLIENTID')]) {
              ansiblePlaybook credentialsId: '496c76de-ea13-4d0f-a945-fec687b54851', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG} aksrg=${aksrg} akscls=${akscls}  AZURECLIENTID=${AZURECLIENT_ID}  AZURECLIENTSECRET=${AZURECLIENTSECRET}  AZURETENANTID=${AZURETENANTID}  AZURESUBSCRIPTIONID=${AZURESUBSCRIPTIONID}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
			  }
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
