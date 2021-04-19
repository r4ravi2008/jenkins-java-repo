
pipeline {
    agent {
        kubernetes {
            // Use a dynamic pod name because static labels are known to cause pod creation errors.
            label "maven-pod-${UUID.randomUUID().toString()}"
            defaultContainer "jnlp"
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: zoltar-batch-offline
spec:
  containers:
  - name: maven
    image: 'maven:3.5.3-jdk-8'
    command:
    - cat
    tty: true
  - name: jnlp
    image: 'jnlp-slave-with-docker:3.26-1_jenkins-2-138-update_3'
    volumeMounts:
    - name: docker-volume
      mountPath: /var/run/docker.sock
  volumes:
    - name: docker-volume
      hostPath:
        path: /var/run/dind/docker.sock
"""
        }
     }
    environment {
        AUTHOR_EMAIL = sh(script: "git log --format='%ae' HEAD^!", returnStdout: true).trim()
    }
    options {
        // timestamps()
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '50', artifactDaysToKeepStr: '30', artifactNumToKeepStr: '50'))
    }
    stages {
        stage('build-test') {
            steps {
                container('maven') {
                    sh '''
                        export M3_HOME=${MAVEN_HOME}/bin
                        mvn clean install
                    '''
                }
            }
        }

    }
    post {
        success {
            echo "Java repo build Success"
            mail to: "${AUTHOR_EMAIL}",
                 subject: "Build Success: ${currentBuild.fullDisplayName}",
                 body: "Build was Successful : ${env.BUILD_URL}"
        }
        failure {
            echo "Java repo build Failure"
            script {
                if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'release/*') {
                    emailext (
                        to: env.SLACK_EMAIL,
                        subject: "Build failure for branch ${env.BRANCH_NAME}: Needs atention",
                        body: "Build failure for branch ${env.BRANCH_NAME}. URL ${env.BUILD_URL}",
                        attachLog: false,
                        )
                }
            }
            mail to: "${AUTHOR_EMAIL}",
                    subject: "Failed build: ${currentBuild.fullDisplayName}",
                    body: "Something is wrong in ${env.BUILD_URL}"
        }
    }
}
