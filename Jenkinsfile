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
    parameters {
        choice(
            name: 'buildOnly',
            choices: 'no\nyes',
            description: 'This will only build the application'
        )
        choice(
            name: 'dockerpush',
            choices: 'no\nyes',
            description: 'This will only push docker image to registry'
        )
        choice(
            name: 'deployToDev',
            choices: 'no\nyes',
            description: 'This will only deploy application to dev environment'
        )
        choice(
            name: 'deployToTest',
            choices: 'no\nyes',
            description: 'This will only deploy application to test environment'
        )
        choice(
            name: 'deployToProd',
            choices: 'no\nyes',
            description: 'This will only deploy application to prod environment'
        )
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
                       script {
                         try {
                         sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker stop ${env.APPLICATION_NAME}-dev\""
                         sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker rm ${env.APPLICATION_NAME}-dev\""
                         }
                         catch(err) {
                             echo "error caught: $err"
                         }
                       }
               // some block
               sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --name ${APPLICATION_NAME}-dev -p 5761:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT \""

                }
            }
        }
        stage('deployingtotest'){
            steps {
                echo "****deploying to test****"
                withCredentials([usernamePassword(credentialsId: 'sudha_docker_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                      script {
                        try {
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker stop ${env.APPLICATION_NAME}-test\""
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker rm ${env.APPLICATION_NAME}-test\""
                        }
                        catch(err) {
                            echo "error caught: $err"
                        }
                      }
               // some block
               sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --name ${APPLICATION_NAME}-test -p 6761:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT \""

                }
            }
            } 
        stage('deployingtoprod'){
            steps {
                echo "****deploying to prod****"
                withCredentials([usernamePassword(credentialsId: 'sudha_docker_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                      script {
                        try {
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker stop ${env.APPLICATION_NAME}-prod\""
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker rm ${env.APPLICATION_NAME}-prod\""
                        }
                        catch(err) {
                            echo "error caught: $err"
                        }
                      }
               // some block
               sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --name ${APPLICATION_NAME}-prod -p 8761:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT \""

                }
            }
        }
        
    }
}