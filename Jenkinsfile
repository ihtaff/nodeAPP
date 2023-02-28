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
      environment {
          BUILD_ID = "${env.BUILD_ID}"
          PACKAGE_FILE = "package.json"
          NEW_VERSION = "${BUILD_ID}.0.0"
      }
      steps {
      
        sh'sed -i 's/version: .*/version: 54.0.0/g' package.json'
        sh 'npm build'
        sh'npm pack'
      }
    }
    stage('Publish NPM Artifact') {
      environment {
    NEXUS_CREDENTIALS = credentials('nexus')
    NEXUS_USERNAME = "${NEXUS_CREDENTIALS_USR}"
    NEXUS_PASSWORD = "${NEXUS_CREDENTIALS_PSW}"
    ARTIFACT_NAME = "nodejs-app-${BUILD_ID}.tgz"
  }
    steps {
        sh '''
       
            curl -u $NEXUS_USERNAME:$NEXUS_PASSWORD -X POST "http://172.17.0.1:8081/service/rest/v1/components?repository=npm-hosted" \
            -H "accept: application/json" \
            -H "Content-Type: multipart/form-data" \
            -F "npm.asset=@${ARTIFACT_NAME};type=application/x-compressed"
        '''
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
