pipeline {
    agent {
        label "k8s-slave"
    }

    tools {
        maven "maven-3.8.8"
        jdk "JDK-17"
    }

    parameters {
        choice(name: 'buildOnly', choices: 'no\nyes', description: 'MVN build application')
        choice(name: 'scanOnly', choices: 'no\nyes', description: 'SonarQube scan app')
        choice(name: 'dockerbuildandpush', choices: 'no\nyes', description: 'dockerbuildandpush')
        choice(name: 'deploytodev', choices: 'no\nyes', description: 'deploy to dev')
        choice(name: 'deploytotest', choices: 'no\nyes', description: 'deploy to test')
        choice(name: 'deploytostage', choices: 'no\nyes', description: 'deploy to stage')
        choice(name: 'deploytoprod', choices: 'no\nyes', description: 'deploytoprod')
    }

    environment {
        APPLICATION_NAME = 'eureka'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = 'docker.io/kishoresamala84'
        DOCKER_CREDS = credentials("kishoresamala84_docker_creds")
    }

    stages {
        stage (' ****** BUILD STAGE ***** ' ) {
            when {
                expression {
                    params.buildOnly == 'yes'
                }
            }
            steps {
                script {
                    buildApp().call()
                }

            }
        }

        stage (' ***** SONARQUBE STAGE ***** ') {
            steps {
                script {
                echo " ***** SONARQUBE STAGE ***** "
                    withSonarQubeEnv('sonarqube') {
                        sh """
                         mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=i27-eureka2 \
                        -Dsonar.host.url=http://34.48.14.175:9000 \
                        -Dsonar.login=sqa_1770f1190375e8cf9d65df9b102c70d43ff4991b 
                        """                   
                    }                    
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage (' ***** BUILD FORMAT ***** ') {
            steps {
                script {
                     sh "echo SOURCE JAR file i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                     sh "echo TARGER JAR file i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                }
            }
        }

        stage (' ***** Docker-Build-Push ***** ') {
            when {
                expression {
                    params.buildOnly == 'yes'
                    params.scanOnly == 'yes'
                    params.dockerbuildandpush == 'yes'
                }
            }
            steps {
                script {
                  dockerBuildPush().call()
                }
            }
        }

        stage (' ***** Deploy to DEV-ENV ***** ') {
            when {
                expression {
                    params.deploytodev == 'yes'
                }
            }
            steps {
                script {
                    deployToDocker('dev','5001','8761').call()
                }
            }
         }

         stage (' ***** Deploy to TEST-ENV ***** ') {
            when {
                expression {
                    params.deploytotest == 'yes'
                }
            }
            steps {
                script {
                    deployToDocker('test','5002','8761').call()
                }
            }
         }

         stage (' ***** Deploy to STAGE-ENV ***** ') {
            when {
                expression {
                    params.deploytostage == 'yes'
                }
            }
            steps {
                script {
                    deployToDocker('stage','5003','8761').call()
                }
            }
         }

         stage (' ***** Deploy to PROD-ENV ***** ') {
            when {
                expression {
                    params.deploytoprod == 'yes'
                }
            }
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
      
    }
}
