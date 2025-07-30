pipeline {
    agent {
        label 'jenkinsslave'
    }
    tools {
        maven 'maven-3.8.9'
    }
    environment {
        APPLICATION_NAME = 'eureka'
        SONAR_URL = "http://34.136.103.101:9000"
        SONAR_TOKEN = credentials"sonar_creds"
    }
    stages {
        stage('build'){
            steps {
                echo "deploying build ${env.APPLICATION} application"
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('sonar'){
            steps {
                echo "sonar scans"
                sh """
                   mvn clean verify sonar:sonar \
                       -Dsonar.projectKey=i27-eureka-sonar \
                       -Dsonar.host.url=${env.SONAR_URL}\
                       -Dsonar.login=${env.SONAR_TOKEN}
                """
            }
        }
    }
}