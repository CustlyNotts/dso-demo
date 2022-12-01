pipeline {
  environment {
    ARGO_SERVER = '35.224.29.73:32100'
  }

  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
	stage('OCI Image BnP') {
  	  steps {
    	    container('kaniko') {
      		sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/custlynotts/dsodemo'
            }	
          }
      	}
      }
    }

    stage('Deploy to Dev') {
      environment {
        AUTH_TOKEN = credentials('argocd-jenkins-deployer-token')
      }
      steps {
        container ('docker-tools') {
          sh 'docker run -t schoolofdevops/argocd-cli argocd app sync dso-demo --insecure --server $ARGO_SERVER --auth-token $AUTH_TOKEN'
          sh 'docker run -t schoolofdevops/argocd-cli argocd app wait dso-demo --health --timeout 300 --insecure --server $ARGO_SERVER --auth-token $AUTH_TOKEN
      }
    }
  }
}
