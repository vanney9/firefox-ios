pipeline {
    agent any
    triggers {
        cron('H 0 * * *')
    }
    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
    }
    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }
        stage('bootstrap') {
            steps {
                sh './bootstrap.sh'
            }
        }
        stage('test') {
            steps {
                dir('SyncIntegrationTests') {
                    sh 'pipenv install'
                    sh 'pipenv check'
                    sh 'pipenv run pytest ' +
                        '--color=yes ' +
                        '--junit-xml=results/junit.xml ' +
                        '--html=results/index.html'
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts 'SyncIntegrationTests/results/*'
            junit 'SyncIntegrationTests/results/*.xml'
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'SyncIntegrationTests/results',
                reportFiles: 'index.html',
                reportName: 'HTML Report'])
        }
        failure {
            slackSend(
                color: 'danger',
                message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        fixed {
            slackSend(
                color: 'good',
                message: "FIXED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
