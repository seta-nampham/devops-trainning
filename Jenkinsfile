pipeline {
   agent none
   environment {
        ENV = "dev"
        NODE = "Build-server"
    }

   stages {
    stage('Build Image') {
        agent {
            node {
                label "Build-server"
                customWorkspace "/home/build/jenkins/"
                }
            }
        environment {
            TAG = sh(returnStdout: true, script: "git rev-parse -short=10 HEAD | tail -n +2").trim()
        }
         steps {
            sh "docker build . -t devops-training-nodejs-$ENV:latest --build-arg BUILD_ENV=$ENV -f Dockerfile"


            sh "cat docker.txt | docker login -u babacaca1103 --password-stdin"
            // tag docker image
            sh "docker tag devops-training-nodejs-$ENV:latest babacaca1103/devops-repo:$TAG"

            //push docker image to docker hub
            sh "docker push babacaca1103/devops-repo:$TAG"

	    // remove docker image to reduce space on build server	
            sh "docker rmi -f babacaca1103/devops-repo:$TAG"

           }
         
       }
	  stage ("Deploy ") {
	    agent {
        node {
            label "Target-Server"
                customWorkspace "/home/namhd/target_data-$ENV/"
            }
        }
        environment {
            TAG = sh(returnStdout: true, script: "git rev-parse -short=10 HEAD | tail -n +2").trim()
        }
	steps {
            sh "sed -i 's/{tag}/$TAG/g' /home/ubuntu/target_data-$ENV/docker-compose.yaml"
            sh "docker-compose up -d"
        }      
       }
   }
    
}