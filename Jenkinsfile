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
    stage('Publish NPM Artifact') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'admin', passwordVariable: 'Kontira@@2022')]) {
            sh "npm config set registry http://172.17.0.1:8081/repository/npm-hosted/"
            sh "npm login --registry=http://172.17.0.1:8081/repository/npm-hosted/ --scope=@my-scope --always-auth"
            sh "npm publish nodejs-app-0.0.0.tgz --registry=http://172.17.0.1:8081/repository/npm-hosted/"
        }
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
