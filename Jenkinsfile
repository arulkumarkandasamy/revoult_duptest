pipeline {
  agent {
       docker {
          image 'arulkumar1967/build-arul-container:latest'
          args '-u root:sudo -v $HOME/workspace/revoultduptest:/revoultduptest'
        }
    }
  environment {
        EMAIL_RECIPIENTS = 'arulkr1967@gmail.com'
  }
  stages {

    stage('Build') {
      steps {
        dir("${JENKINS_HOME}/workspace/revoultduptest/") {
          sh 'mvn package'
          sh 'rm -f packer/bin/*.jar'
          sh 'cp -r target/*.jar packer/bin'
        }
        /* withAWS(endpointUrl:'https://s3.amazonaws.com', credentials:'ada90a34-30ef-47fb-8a7f-a97fe69ff93f'){
			s3Upload(file:'gs-spring-boot-0.1.0.jar', bucket:'arulrevoulttest', path:'revoultduptest/target/gs-spring-boot-0.1.0')
		} */
      }
    }
    stage('Create Packer AMI') {
        steps {
          withCredentials([
            usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
          ]) {
            sh 'packer build -var aws_access_key=${AWS_KEY} -var aws_secret_key=${AWS_SECRET} packer/packer.json'
          }
       }
    }
    stage('AWS Deployment') {
      steps {
          withCredentials([
            usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
            usernamePassword(credentialsId: '2facaea2-613b-4f34-9fb7-1dc2daf25c45', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER'),
          ]) {
            sh '''
               cd terraform
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
            '''
        }
      }
    }
  }
  
  }
