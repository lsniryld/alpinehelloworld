pipeline{
	environment {
		IMAGE_NAME= "alpinehelloworld"
        IMAGE_TAG= "latest"
    	STAGING = "niry-staging"
    	PRODUCTION = "niry-production"
		ID_DOCKER = "${ID_DOCKER_PARAMS}"
		APP_NAME = "nini"
		STG_API_ENDPOINT = "http://ip10-0-3-4-cedmof4iqmmgg4teoer0-1993.direct.docker.labs.eazytraining.fr/"
		STG_APP_ENDPOINT = "http://ip10-0-3-3-cedmof4iqmmgg4teoer0-80.direct.docker.labs.eazytraining.fr/"
		PROD_API_ENDPOINT = "http://ip10-0-3-4-cedmof4iqmmgg4teoer0-1993.direct.docker.labs.eazytraining.fr/"
		PROD_APP_ENDPOINT = "http://ip10-0-3-4-cedmof4iqmmgg4teoer0-80.direct.docker.labs.eazytraining.fr/"
		INTERNAL_PORT = "5000"
		EXTERNAL_PORT = "${PORT_EXPOSED}"
		CONTAINER_IMAGE = "${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG"

    }
    agent none
  	stages{
		stage('Build image'){
			agent any
			steps {
				script{
					sh 'docker build -t $ID_DOCKER/$IMAGE_NAME:$IMAGE_TAG .'
					}
			}
		}
		stage('Run container based on builded image'){
			agent any
			steps {
				script{
					sh '''
					echo "Clean environment"
					docker rm -f $IMAGE_NAME || echo "Container does not exist"
					docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:${INTERNAL_PORT} -e PORT=${INTERNAL_PORT} ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
					sleep 5
					'''
				}
			}
		}
		stage('Test image'){
			agent any
			steps {
				script{
					sh '''
					curl $STG_APP_ENDPOINT | grep -q "Hello"
					'''
				}
			}
		}
		stage('Clean container'){
			agent any
			steps {
				script{
					sh '''
					docker stop $IMAGE_NAME 
					docker rm $IMAGE_NAME
					'''
				}
			}
		}
		stage ('Login and Push Image on docker hub') {
          	agent any
          	environment {
           		DOCKERHUB_PASSWORD  = credentials('dockerhub')
          	}  
          	steps {
             	script {
               		sh '''
                   		echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
                   		docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               		'''
             	}
          	}
        }   
		stage('STAGING - Deploy app'){	
			when {
				expression { GIT_BRANCH == 'origin/master' }
			}
			agent any
			steps {
				script{
					sh """
					echo{\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\",
   					\\"external_port\\":\\"${EXTERNAL_PORT}\\",\\"internal_port\\":\\"${INTERNAL_PORT}\\"} > data.json
   
   					curl -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json' --data-binary @data.json
					"""
				}
			}
		}
  }
}
