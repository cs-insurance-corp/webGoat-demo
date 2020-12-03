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
        script{
        def version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
        }
        sh "echo ${version}"

      }
        junit '**/target/surefire-reports/TEST-*.xml'

      }
    }
    // stage('IQServer') {
    //   // when { 
    //   //   branch 'main'
    //   //   beforeAgent true
    //   // }
    //   agent {
    //     label 'iq-server-cli-pod'
    //   }
    // steps {
    //  container('iq-server-cli') {
    //     //sleep time: 10, unit: 'MINUTES'
    //     checkout scm
    //     sh '''
    //     mvn clean install -s /usr/share/maven/ref/settings.xml
    //     '''
    //   }
    //     junit '**/target/surefire-reports/TEST-*.xml'

    //   }
    // }
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