#!/usr/bin/env groovy
pipeline {
  agent none
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipDefaultCheckout true
  }
  stages {
    stage('Maven Build WebGoat') {
      when { 
       not {
           branch 'release-*'
       }
        beforeAgent true
      }
      agent {
        kubernetes{
          label 'maven'
          yamlFile 'kubernetes-agents/mvn.yaml'
        }
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
      when { 
       not {
           branch 'release-*'
       }
        beforeAgent true
      }
      agent {
        kubernetes{
          label 'iqserver-cli'
          yamlFile 'kubernetes-agents/iqserver-cli.yaml'
        }
      }
    steps {
     container('iq-cli') {
       unstash 'webgoat-server-jars'
      //  sh 'ls -l -R'
       withCredentials([usernameColonPassword(credentialsId: 'credentials-iq-server', variable: 'iqserver')]) {
       sh '/sonatype/evaluate -s http://35.237.47.88:8070 -i webGoat-demo -a $iqserver ./webgoat-server/target/*.jar'
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
        kubernetes{
          label 'maven'
          yamlFile 'kubernetes-agents/mvn.yaml'
        }
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
        kubernetes{
          label 'maven'
          yamlFile 'kubernetes-agents/mvn.yaml'
        }
      }
    steps {
     container('maven') {
        //sleep time: 10, unit: 'MINUTES'
        checkout scm
        sh '''
        mvn clean install deploy -Drevision=${BRANCH_NAME#*-} -s /usr/share/maven/ref/settings.xml
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
        kubernetes{
          label 'hub-cli'
          yamlFile 'kubernetes-agents/hub-cli.yaml'
        }
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