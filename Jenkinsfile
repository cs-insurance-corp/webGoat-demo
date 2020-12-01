pipeline {
  agent {
    kubernetes {
      label 'mvn-pod'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    label: mvn-pod
spec:
  containers:
  - name: maven
    image: maven:3.6-jdk-11-openj9
    command:
    - cat
    tty: true
"""
    }
  }
  stages {
    stage('Maven') {
      steps {
     container('maven') {
       withCredentials([file(credentialsId: 'settings-xml', variable: 'settings-xml')]) {
        sh '''
        mvn clean install -s ${settings-xml}
        '''
        }
      }
      }
    }
  }
}