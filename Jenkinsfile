#!/usr/bin/env groovy
pipeline {
  agent none
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipDefaultCheckout true
  }

  stages {
    stage('Maven Build WebGoat') {
      // when { 
      //   branch 'main'
      //   beforeAgent true
      // }
      agent {
        label 'mvn-pod'
      }
    steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean install -s /usr/share/maven/ref/settings.xml
        '''
      }
        junit '**/target/surefire-reports/TEST-*.xml'

      }
    }
    stage('Push WebGoat to Snapshot Repository') {
      // when { 
      //   branch 'main'
      //   beforeAgent true
      // }
      agent {
        label 'mvn-pod'
      }
    steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean deploy -Dmaven.test.skip=true -s /usr/share/maven/ref/settings.xml
        '''
      }
      }
    }
    stage('Release WebGoat') {
      // when { 
      //   // branch 'release-*'
      //   beforeAgent true
      // }
      agent {
        label 'mvn-pod'
      }
    steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean deploy scm:tag -Dmaven.test.skip=true -s /usr/share/maven/ref/settings.xml
        '''
      }
      }
    }   
  }
}