pipeline {
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipDefaultCheckout true
  }
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
      - mountPath: "/root/.m2"
        name: mvn-pv-storage
  volumes:
    - name: settings-xml
      secret:
        secretName: settings-xml
    - name: mvn-pv-claim
      persistentVolumeClaim:
        claimName: mvn-pv-claim

"""
    }
  }
  stages {
    stage('Maven') {
      steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean install -s /usr/share/maven/ref/settings.xml
        '''
      }
      }
    }
  }
}