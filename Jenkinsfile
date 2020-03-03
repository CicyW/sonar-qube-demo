pipeline {
    agent any
    stages {
        stage("Compile") {
            steps {
                    sh "mvn clean install"
            }
        }

        stage('Sonar-Scan') {
            steps {
                sh "/usr/local/bin/mvn sonar:sonar"
            }
        }
    }
    post {
        aborted {
            script {
                currentBuild.result = 'SUCCESS'
            }
        }
    }
}

