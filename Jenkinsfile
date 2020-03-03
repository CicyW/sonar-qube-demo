pipeline {
    agent { docker 'maven:3.3.3' }
    stages {
        stage("build") {
            steps {
                    sh "mvn clean install"
            }
        }

        stage('Sonar-Scan') {
            steps {
                sh '''
                BRIDGES_IP=`/sbin/ip route|awk '/default/ { print $3 }'`
                mvn sonar:sonar \
                -Dsonar.host.url=http://${BRIDGES_IP}:9000 \
                -Dsonar.login=admin  \
                -Dsonar.password=admin
                '''
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
