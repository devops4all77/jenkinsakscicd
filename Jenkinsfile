pipeline{
    agent any
    environment {
      DOCKER_TAG = getVersion()
	  aks_rg = "sprinrg"
	  aks_cls = "springaks"
	  acrname = "acrjen1122443.azurecr.io"
	  acr_username = "acrjen1122443"
	  AZURE_TENANT_ID = "0c508b0d-c4b2-4a38-93c7-5947fb5a5656"
	  AZURE_SUBSCRIPTION_ID = "e44c9379-2fc4-4ec0-8945-b7fd3ff798bd"
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
               withCredentials([usernamePassword(credentialsId: 'spauth', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                    sh "sudo docker login ${acrname} -u ${acr_username}  -p ${AZURE_CLIENT_SECRET}"
                }
              
                sh "sudo docker push ${acrname}/helloapp:latest "
            }
        }
        
        stage('Deploy'){
            steps{
			withCredentials([usernamePassword(credentialsId: 'spauth', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
              ansiblePlaybook credentialsId: '496c76de-ea13-4d0f-a945-fec687b54851', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG} aks_rg=${aks_rg} aks_cls=${aks_cls} AZURE_CLIENT_ID=${AZURE_CLIENT_ID} AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET} AZURE_TENANT_ID=${AZURE_TENANT_ID} AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
			  }
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
