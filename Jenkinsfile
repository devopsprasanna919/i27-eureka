// This pipeline is for Eureka microservice deployment
pipeline {
    agent {
        label 'java-slave'
    }

    // tools section 
    tools {
        maven 'maven-3.8.9'
        jdk 'JDK-17'
    }

    // environment 
    environment {
        // read the pom.xml file 
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        APPLICATION_NAME = "eureka"
        SONAR_URL = "http://34.30.23.85:9000"
        SONAR_TOKEN = credentials('sonar_creds')
    }

    stages {
        stage ('Build'){
            steps {
                // using maven
                echo "********** Building ${env.APPLICATION_NAME} Application *************"
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage ('Sonar') {
            steps {
                echo "************** Starting Sonar Scan **************"
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=${env.SONAR_URL}\
                            -Dsonar.login=${env.SONAR_TOKEN}
                    """
                }
                timeout (time: 2, unit: "MINUTES") {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }
                 





            }
        }
        stage ('FormatBuild'){
            // existing i27-eureka-0.0.1-SNAPSHOT.jar
            // Destination i27-eureka-buildnumber-brnachname.jar
            steps {
                echo "Testing JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "Testing JAR Destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
            }
        }
        stage ('DockerBuildAndPush') {
            steps {
                echo "************* Building the Docker image ***************"
                sh "cp target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t eureka:v4 ./.cicd"
            }
        } // i27-eureka-0.0.1-SNAPSHOT.jar
    }
}