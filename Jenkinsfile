pipeline {
    options {
      disableConcurrentBuilds()
    }  
    agent {
      label "jenkins-maven-java11"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'ttc-example'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"
      HELM_RELEASE_NAME = "ttcexample-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()

      PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
      PREVIEW_NAMESPACE = "ttcexample-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()
      
      REALM = "activiti"

    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE.35.197.207.143.nip.io"
         SSO_HOST = "activiti-keycloak.$PREVIEW_NAMESPACE.35.197.207.143.nip.io"
        }
        steps {
          container('maven') {
           dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install'
            }

            dir("./ttc-acceptance-tests") {
              git 'https://github.com/Activiti/ttc-acceptance-tests.git'
              sh 'sleep 190'
              sh "mvn clean verify"
            }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
	environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE.35.197.207.143.nip.io"
         SSO_HOST = "activiti-keycloak.$PREVIEW_NAMESPACE.35.197.207.143.nip.io"
        }      
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
            dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install'
            }
	   //run tests	
            dir("./ttc-acceptance-tests") {
              git 'https://github.com/Activiti/ttc-acceptance-tests.git'
              sh 'sleep 185'
              sh "mvn clean verify"
            }	  
	    //end run tests
          }
        }
      }

    }
   post {
        always {
          container('maven') {
            dir("./charts/$APP_NAME") {
               sh "make delete" 
            }
            sh "kubectl delete namespace $PREVIEW_NAMESPACE" 
          }
          cleanWs()
        }
   }
}
