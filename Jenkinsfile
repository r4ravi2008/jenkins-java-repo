pipeline {
    agent {
        kubernetes {
            // Use a dynamic pod name because static labels are known to cause pod creation errors.
            label "maven-pod-${UUID.randomUUID().toString()}"
            defaultContainer "maven"
            yamlFile 'pods.yaml'
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

        // stage ('deploy') {
        //    // public url
        // }

        stage ('zap-scan') {
            post {
                always {
                    container ('zap') {
                        sh '''
                        zap-cli --verbose report -o ./owasp-quick-scan-report.html --output-format html
                        ls -lah
                        '''
                    }
                    publishHTML target: [
                        allowMissing         : false,
                        alwaysLinkToLastBuild: false,
                        keepAll              : true,
                        reportDir            : './',
                        reportFiles          : 'owasp-quick-scan-report.html',
                        reportName           : 'Zap Scan Report'
                    ]
                }
            }
            steps {
                container('zap') {
                    // Update the target and scan types as per your needs.
                    // Documentation here: https://www.zaproxy.org/docs/docker/about/
                   sh '''
                       zap-cli --verbose quick-scan http://www.itsecgames.com
                   '''

                }
            }
        }

    }
    // post {
    //     success {
    //         echo "Java repo build Success"
    //         mail to: "${AUTHOR_EMAIL}",
    //              subject: "Build Success: ${currentBuild.fullDisplayName}",
    //              body: "Build was Successful : ${env.BUILD_URL}"
    //     }
    //     failure {
    //         echo "Java repo build Failure"
    //         script {
    //             if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'release/*') {
    //                 emailext (
    //                     to: env.SLACK_EMAIL,
    //                     subject: "Build failure for branch ${env.BRANCH_NAME}: Needs atention",
    //                     body: "Build failure for branch ${env.BRANCH_NAME}. URL ${env.BUILD_URL}",
    //                     attachLog: false,
    //                     )
    //             }
    //         }
    //         mail to: "${AUTHOR_EMAIL}",
    //                 subject: "Failed build: ${currentBuild.fullDisplayName}",
    //                 body: "Something is wrong in ${env.BUILD_URL}"
    //     }
    // }
}
