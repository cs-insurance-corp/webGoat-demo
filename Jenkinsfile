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
    volumeMounts:
      - mountPath: "/usr/share/maven/ref"
        name: settings-xml
        readOnly: true
  volumes:
    - name: settings-xml
      secret:
        secretName: settings-xml
"""
    }
  }
  stages {
    stage('Maven') {
      steps {
     container('maven') {
        sh '''
        mvn clean install 
        '''
      }
      }
    }
  }
}