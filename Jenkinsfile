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
            withMaven(serviceAccount: "jenkins", mavenSettingsXmlSecret: "maven-secret") {
                        container('maven') {
                            sh "mvn -version"
                        }
                    }
                }
        }
    }
}
