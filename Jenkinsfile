pipeline {
    agent { label 'JDK8'}
    environment {
        MVN = '/usr/share/maven/bin/mvn'
    }
    options { timeout(time: 1, unit: 'HOURS') }
    triggers { cron('0 * * * *') }
    stages {
        stage('git checkout') {
            steps {
                checkout scm
            }
        }
        stage('code compilation') {
            steps {
                sh '$MVN clean package'
            }
        }
        stage('archive artifact and junit test report') {
            steps {
                junit testResults : 'target/surefire-reports/*.xml'
            }
        }

    }
    post { 
        always { echo "pipeline is running" }
        success { echo "success" }
        unsuccessful { echo "yet to do changes" }
    }
}
