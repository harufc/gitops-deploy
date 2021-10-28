pipeline {
    environment {
        DOMAIN='apps.ocp4.example.com'
        PRJ="hello-${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        APP='nodeapp'
    }
    agent {
      node {
        label 'nodejs'
      }
    }
    stages {
        stage('create') {
            steps {
                script {
                    // Uncomment to get lots of debugging output
                    //openshift.logLevel(1)
                    openshift.withCluster() {
                        echo("Create project ${env.PRJ}")
                        openshift.newProject("${env.PRJ}")
                        openshift.withProject("${env.PRJ}") {
                            echo('Grant to developer read access to the project')
                            openshift.raw('policy', 'add-role-to-user', 'view', 'developer')
                            echo("Create app ${env.APP}")
                            openshift.newApp("${env.GIT_URL}#${env.BRANCH_NAME}", "--strategy source", "--name ${env.APP}")
                        }
                    }
                }
            }
        }
        stage('build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${env.PRJ}") {
                            def bc = openshift.selector('bc', "${env.APP}")
                            echo("Wait for build from bc ${env.APP} to finish")
                            timeout(5) {
                                def builds = bc.related('builds').untilEach(1) {
                                    def phase = it.object().status.phase
                                    if (phase == "Failed" || phase == "Error" || phase == "Cancelled") {
                                        error 'OpenShift build failed or was cancelled'
                                    }
                                    return (phase == "Complete")
                                }
                            }
                        }
                    }
                }
            }
        }
