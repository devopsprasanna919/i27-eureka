pipeline {
    agent {
        label 'jenkinsslave'
    }
    tools {
        maven 'maven-3.8.9'
    }
    environment {
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        APPLICATION_NAME = 'eureka'
        SONAR_URL = "http://34.136.103.101:9000"
        SONAR_TOKEN = credentials"sonar_creds"
        DOCKER_HUB = "docker.io/sudhadevops8"
        DOCKER_CREDS = credentials('docker_hub_creds')
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
                withSonarQubeEnv('SonarQube') {
                    sh """
                    mvn clean verify sonar:sonar \
                       -Dsonar.projectKey=i27-eureka-sonar \
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
        stage('dockerbuildandpush'){
            steps {
                echo "*** running docker build ***"
                sh "cp target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT ./.cicd"
                echo "****Docker Login****"
                sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                echo "****Docker Push ******"
                sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT"
            }
        }
        stage('deployingtodev'){
            steps {
                echo "****deploying to dev****"
                withCredentials([usernamePassword(credentialsId: 'sudha_docker_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
               // some block
               sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"hostname -i\""
               sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --name ${APPLICATION_NAME}-dev -p 5761:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT \""

                }

            }
        }
    }
}