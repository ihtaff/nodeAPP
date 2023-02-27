pipeline {
  agent any 
  
  tools {
    nodejs 'node'
  }
  
  environment {
    NPM_REGISTRY = 'http://172.17.0.1:8081/repository/npm-proxy'
  }
  
  stages {
    stage ('Initialize') {
      steps {
        sh '''
          echo "PATH = ${PATH}"
          echo "NODE_HOME = ${NODE_HOME}"
        ''' 
      }
    }
    stage('Code checkout') {
      steps {
        checkout scmGit(branches: [[name: '**']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ihtaff/nodeAPP.git']])
      }
    }
    stage('Configure npm registry') {
      steps {
        sh "npm config set registry ${NPM_REGISTRY}"
      }
    }
    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }
    stage('Build') {
      steps {
        sh 'npm build'
      }
    }
    stage ('Source Composition Analysis') {
      steps {
        dependencyCheck additionalArguments: '--format HTML --format XML --format JSON', odcInstallation: 'DC'
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
  }
  
      post {
        always {
        echo 'One way or another, I have finished'
        deleteDir() /*IMPORTANT FOR ALL PIPELINES! clean up our workspace, to avoid saturating the Jenkins server storage*/
        }
         success {
            echo 'I succeeeded!'
        }
         unstable{
            echo 'I am unstable :/'
         } 
         failure {
            echo 'I failed :('
         }
    }
}
