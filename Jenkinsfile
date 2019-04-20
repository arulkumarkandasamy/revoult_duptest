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
        }
        withCredentials([
            usernamePassword(credentialsId: 'ada90a34-30ef-47fb-8a7f-a97fe69ff93f', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
          ]) {
            sh 's3Upload(file:'gs-spring-boot-0.1.0.jar', bucket:'arulrevoulttest', path:'${JOB_NAME}-${BUILD_NUMBER}/target/gs-spring-boot-0.1.0.jar')'
        }

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
            sh 'rm -rf revtest_practice1'
            sh 'git clone https://github.com/arulkumarkandasamy/revtest_practice1.git'
            sh '''
               cd revtest_practice1
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="Arulkumar Kandasamy" -c user.email="karul43@yahoo.co.in" commit -m "terraform state update from Jenkins"
               git push https://${REPO_USER}:${REPO_PASS}@github.com/arulkumarkandasamy/revtest_practice1.git master
            '''
        }
      }
    }
  }
  
  }
