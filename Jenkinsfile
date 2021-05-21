library identifier: "pipeline-library@v1.5",
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/redhat-cop/pipeline-library.git"
  ]
)

// The name you want to give your Spring Boot application
// Each resource related to your app will be given this name
appName = "myapp-java-spring-boot-pipeline"

openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  echo "Starting Pipeline for ${env.APP_NAME}..."
  env.BUILD = "${env.NAMESPACE_BUILD}"
  env.DEV = "${env.NAMESPACE_DEV}"
  env.MAVEN_SETTINGS = "${env.MAVEN_SETTINGS}"
  env.MAVEN_SERVER_USERNAME = "${env.MAVEN_SERVER_USERNAME}"
  env.MAVEN_SERVER_PASSWORD = "${env.MAVEN_SERVER_PASSWORD}"
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

        // Run Maven build, skipping tests
        stage('Build'){
          steps {
            sh "mvn -B clean install -DskipTests=true -f ${POM_FILE} -s ${MAVEN_SETTINGS}"
          }
        }

        // Run Maven unit tests
        stage('Unit Test'){
          steps {
            sh "mvn -B test -f ${POM_FILE} -s ${MAVEN_SETTINGS}"
          }
        }

        // Build Container Image using the artifacts produced in previous stages
        stage('Build Container Image'){
          steps {
            // Copy the resulting artifacts into common directory
            sh """
              ls target/*
              rm -rf oc-build && mkdir -p oc-build/deployments
              for t in \$(echo "jar;war;ear" | tr ";" "\\n"); do
                cp -rfv ./target/*.\$t oc-build/deployments/ 2> /dev/null || echo "No \$t files"
              done
            """

            // Build container image using local Openshift cluster
            // Giving all the artifacts to OpenShift Binary Build
            // This places your artifacts into right location inside your S2I image
            // if the S2I image supports it.
            binaryBuild(projectName: env.BUILD, buildConfigName: env.APP_NAME, buildFromPath: "oc-build")
          }
        }

//         stage('Promote from Build to Dev') {
//           steps {
//             tagImage(sourceImageName: env.APP_NAME, sourceImagePath: env.BUILD, toImagePath: env.DEV)
//           }
//         }
//
//         stage ('Verify Deployment to Dev') {
//           steps {
//             verifyDeployment(projectName: env.DEV, targetApp: env.APP_NAME)
//           }
//         }
    }
}
