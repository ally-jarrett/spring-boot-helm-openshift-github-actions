library identifier: "pipeline-library@v1.5",
retriever: modernSCM([ $class: "GitSCMSource", remote: "https://github.com/redhat-cop/pipeline-library.git"])

library identifier: "mylibrary@master",
retriever: modernSCM([ $class: "GitSCMSource", remote: "https://github.com/ally-jarrett/pipeline-library-demo.git"])

// The name you want to give your Spring Boot application
// Each resource related to your app will be given this name
appName = "myapp-java-spring-boot-pipeline"
settings = "/etc/config/settings.xml"

openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  echo "Starting Pipeline for ${env.APP_NAME}..."
  env.BUILD = "${env.NAMESPACE_BUILD}"
  env.DEV = "${env.NAMESPACE_DEV}"
}

podTemplate(yaml:'''
spec:
  containers:
  - name: maven
    image: image-registry.openshift-image-registry.svc:5000/openshift/jenkins-agent-maven:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: home-volume
      mountPath: /home/jenkins
    env:
    - name: HOME
      value: /home/jenkins
    - name: MAVEN_OPTS
      value: -Duser.home=/home/jenkins
    - name: MAVEN_SERVER_USERNAME
      valueFrom:
        secretKeyRef:
          name: nexus-secret
          key: username
    - name: MAVEN_SERVER_PASSWORD
      valueFrom:
        secretKeyRef:
          name: nexus-secret
          key: password
  volumes:
    - name: home-volume
      emptyDir: {}
    - name: maven-settings
      configMap:
      name: maven-settings
''')

pipeline {
    // Use the 'maven' Jenkins agent image which is provided with OpenShift
    agent { label "maven" }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage('Demo') {
            steps {
                echo 'Hello world'
                sayHello 'Dave'
            }
        }

        // Run Maven build, skipping tests
        stage('Maven Build'){
            steps {
                container('maven') {
                    sh "mvn -version"
                }
            }
        }
    }
}
