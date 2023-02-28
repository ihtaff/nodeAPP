pipeline {
  agent any 
  
  tools {
    nodejs 'node'
  }
  
  environment {
    NEXUS_LOGIN = "nexus"
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
    stage ('Source Composition Analysis') {
      steps {
        dependencyCheck additionalArguments: '--format HTML --format XML --format JSON', odcInstallation: 'DC'
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
    stage('Build and package') {
      steps {
        sh 'npm build'
        sh'npm pack'
      }
    }
    stage('Upload Artifact to Nexus') {
      steps {
        nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: '172.17.0.1:8081',
          groupId: 'mygroupID',
          version: "${env.BUILD_ID}",
          repository: 'npm-hosted',
          credentialsId: "${NEXUS_LOGIN}",
          artifacts: [
            [artifactId: 'myArtifact',
             classifier: '',
             file: 'nodejs-app-0.0.0.tgz',
             type: 'tgz']
          ]
        )
      }
    }
  }
  

}
