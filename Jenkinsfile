pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'maven-3.8.8'
        jdk 'JDK-17'
    }
    environment {
        APPLICATION_NAME='eureka'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/kishoresamala84"
        DOCKER_CREDS = credentials("kishoresamala_docker_creds")
    }
    stages {
        stage (' ***** BUILD STAGE ***** ') {
            steps {
                echo "*****Building the Application *****"
                sh "mvn clean package -DskipTest=true"
                archiveArtifacts 'target/*.jar'
            }
        }

        stage (' ***** SONARQUBE STAGE ***** ') {
            steps {
                echo "***** sonar stage implementing *****"                
                withSonarQubeEnv('sonarqube') {
                    sh """
                         mvn sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=http://34.48.14.175:9000 \
                            -Dsonar.login=sqa_e27457e7a7bce38fdd73f05e767b4368d7355ee3
                    """
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ('BUILD-FORMAT') {
            steps {
                script {
                    // Existing : i27-eureka-0.0.1-SNAPSHOT.jar
                    // Destination: i27-eureka-buildnumber-branchname.packaging
                    sh """
                    echo "Testing source jar-source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    echo "Tesing destination Jar: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                    """ 
                }
            }
        }

        stage (' ***** Docker-Build-Push ***** ') {
            steps {
                script {
                    echo "****** Building Doker image *******"                    
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd "
                    sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                    echo "****** Building Doker image *******" 
                    sh "docker push ${DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"                    
                }
            }
        }

        stage (' ***** Deploy to DEV-ENV ***** ') {
            steps {
                echo " ***** deploy to dev env ***** "
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_passwd', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        script {    
                            try {
                                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker stop ${env.APPLICATION_NAME}-dev \""
                                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker rm ${env.APPLICATION_NAME}-dev \""
                            }
                            catch(err){
                                echo "Error Caught: $err"                      
                            }  
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME}-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}\""                    
                        }
                   }
               }
           }
       }
   }
