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
        stash includes: 'webgoat-server/target/*SNAPSHOT.jar', name: 'webgoat-server-jars'

      }
        junit '**/target/surefire-reports/TEST-*.xml'

      }
    }
    stage('IQServer') {
      // when { 
      //   branch 'main'
      //   beforeAgent true
      // }
      agent {
        label 'iq-cli'
      }
    steps {
     container('iq-cli') {
       unstash 'webgoat-server-jars'
       sh 'ls -l -R'
       withCredentials([usernameColonPassword(credentialsId: 'credentials-iq-server', variable: 'credentials-iq-server')]) {
       sh ("/sonatype/evaluate -s http://35.237.47.88:8070 -i webGoat-demo -a $credentials-iq-server /*.jar")
       }
      }

      }
    }
    stage('Push Snapshot to Nexus') {
      when { 
       not {
           branch 'release-*'
       }
        beforeAgent true
      }
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
   stage('Pushing Release to Nexus') {
      when { 
        branch 'release-*'
        beforeAgent true
      }
      agent {
        label 'mvn-pod'
      }
    steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean deploy -Dmaven.test.skip=true -Drevision=${BRANCH_NAME#*-} -s /usr/share/maven/ref/settings.xml
        '''
      }
      }
    }   
    stage('Release WebGoat') {
      when { 
        branch 'release-*'
        beforeAgent true
      }
      agent {
        label 'hub-cli'
      }
    steps {
     container('hub') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        hub release create -m "Release 🚀 v${BRANCH_NAME#*-}" ${BRANCH_NAME#*-}
        '''
      }
      }
    }   
  }
}