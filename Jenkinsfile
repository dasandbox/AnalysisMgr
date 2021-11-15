import java.text.SimpleDateFormat
def idtag // dataset/analsys id

pipeline {
    agent any

    options {
        timestamps() // Add timestamps to logging
        timeout(time: 5, unit: 'HOURS') // Abort pipleine

        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
    }
    environment {
        PATH = "/usr/local/bin:$PATH"
    }
    parameters {
        string(name: 'idtag', defaultValue: '', description: 'Enter a custom Analysis Id (No Spaces!) or a default [YYMMDD_HHMMSS] will be created')
        choice(name: 'baseline',
               choices: [
                   'TTWCS_Baseline_5_6_1',
                   'TTWCS_Baseline_5_6_0'
               ],
               description: 'Baseline')
    }

    stages {

        stage('Init') {
            steps {
                echo 'Stage: Init'
                echo "branch=${env.BRANCH_NAME}, params.idtag=${params.idtag}, baseline=${params.baseline}"
                script {
                    if (params.idtag == null || params.idtag == '') {
                        idtag = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date())
                    } else {
                        idtag = params.idtag
                    }
                    echo "Analysis idtag: ${idtag}"
                }
            }
        }
        stage('Common Config') {
            steps {
                echo 'Stage: Common Config'
                echo "idtag: ${idtag}"
                // Checkout repo with common config files/scripts to 'common' folder
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'common']],
                          submoduleCfg: [], userRemoteConfigs: [[url: 'file:///home/jenkins/gitrepos/cicd-common']]
                         ])
            }
        }
        stage('Run Local ETL') {
            steps {
                echo 'Stage: Run Local ETL'
                dir('common') {
                    sh """
                    ls -l
                    ./am_run_local_etl.sh ${idtag} ${params.baseline}
                    """
                }
            }
        }
        stage('Run Local Analysis') {
            steps {
                echo 'Stage: Run Local Analysis'
                dir('common') {
                    sh """
                    ./am_run_local_analysis.sh ${idtag} ${params.baseline}
                    """
                }
            }
        }
        stage('Archive Reports') {
            steps {
                echo 'Stage: Archive Reports'
                // zip (zipFile: "/home/jenkins/WinShare/analysis/${idtag}_reports.zip", dir: "/home/jenkins/WinShare/analysis/${idtag}")
            }            
        }

    }
    post {
        always {
            echo "post/always"
            deleteDir() // clean workspace
        }
        success {
            echo "post/success"
        }
        failure {
            echo "post/failure"
        }
    }
}
