pipeline {
    agent {
        label 'jenkinslave'
    }
    tools {
        maven 'maven-3.8.9'
    }
    stages {
        stage('build'){
            steps {
                echo "deploying build eureka application"
                sh "mvn clean package"
            }
        }
    }
}