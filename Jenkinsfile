pipeline {
	options {
		buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
		//skipDefaultCheckout() 
		disableConcurrentBuilds()
	}
	agent { label 'agent1' }
	parameters {
		booleanParam(name: "Deploy", defaultValue: false, description: "Deploy the Build to EKS cluster")
    }	
    environment {
	ecrRepo = "674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo"
        //ecrCreds = 'awscreds'
	//dockerImage = "674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo:latest"
	dockerImage = "${env.ecrRepo}:${env.BUILD_ID}"
    }
	
    stages{
	    stage('Docker Image Build') {
		    steps {
			    sh 'docker build -t $dockerImage ./docker/'
			    sh 'docker tag $ecrRepo $ecrRepo:latest'
		    }}
	    stage('Push Image to ECR'){
		    steps {
			    script {
				    sh 'docker push $ecrRepo:latest'
				    def exit1 = sh script: 'echo $?'
                                    if (exit1 != 0){
                                    echo $(aws ecr get-authorization-token --region us-east-2 --output text --query 'authorizationData[].authorizationToken') | base64 -d | cut -d: -f2 | docker login -u AWS 674583976178.dkr.ecr.us-east-2.amazonaws.com --password-stdin'
				    sh 'docker push $ecrRepo:latest'
				    }
				    sh 'docker push $dockerImage'
			    }}
		    post { success { sh 'docker builder prune --all -f' } }
	    }
	    stage('Deploy to EKS'){
                 when { expression { return params.Deploy }}
            steps {
		script{
		 sh 'eksctl get cluster --region us-east-2'
		 def exit2 = sh script: 'echo $?'
		 if (exit2 != 0){
			sh './k8s/cluster.sh'
		 }
                 sh '''
		 kubectl apply -f ./k8s/eksdeploy.yml
		 kubectl get deployments  
                 sleep 5
                 kubectl get svc
                 '''   }}
		    post {
			    always { cleanWs() } }
	    }
    }
	post { always { cleanWs() } }
}
