// Build a Maven project using the standard image and Scripted syntax.
// Rather than inline YAML, you could use: yaml: readTrusted('jenkins-pod.yaml')
// Or, to avoid YAML: containers: [containerTemplate(name: 'maven', image: 'maven:3.6.3-jdk-8', command: 'sleep', args: 'infinity')]
podTemplate(yaml: '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.6.3-jdk-8
    command:
    - sleep
    args:
    - infinity
''') {
    node(POD_LABEL) {
        container('maven') {
            git checkout
            sh 'mvn clean install'
        }
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts '**/target/*.jar'
    }
}
