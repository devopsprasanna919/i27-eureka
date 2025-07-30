pipeline {
    agent {
        label 'jenkinsslave'
    }
    tools {
        maven 'maven-3.8.9'
    }
    environment {
        APPLICATION_NAME = 'eureka'
    }
    stages {
        stage('build'){
            steps {
                echo "deploying build ${env.APPLICATION} application"
                sh "mvn clean package -Dskip Tests=true"
            }
        }
        stage('sonar'){
            steps {
                echo "sonar scans"
                sh """
                   mvn clean verify sonar:sonar \
                       -Dsonar.projectKey=i27-eureka-sonar \
                       -Dsonar.host.url=http://34.136.103.101:9000 \
                       -Dsonar.login=sqp_52fcc4853f57765c05c7c0dac047e8a513a28d10
                """
            }
        }
    }
}