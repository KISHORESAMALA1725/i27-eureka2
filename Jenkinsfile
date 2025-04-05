pipeline {
    agent {
        label "k8s-slave"
    }

    tools {
        maven "maven-3.8.8"
        jdk "JDK-17"
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
            steps {
                echo " ***** THIS IS BUILD STAGE ***** "
                sh "mvn clean package -DskipTest=true"
                archiveArtifacts 'target/*.jar'
            }
        }

        stage (' ***** SONARQUBE STAGE ***** ') {
            steps {
                echo " ***** SONARQUBE STAGE ***** "
                withSonarQubeEnv('sonarqube') {
                    sh """
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=i27-eureka2 \
                    -Dsonar.host.url=http://34.48.14.175:9000 \
                    -Dsonar.login=sqa_1770f1190375e8cf9d65df9b102c70d43ff4991b 
                    """                   
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

        stage (' ***** DOCKER BUILD AND PUSH ***** ') {
            steps {
                script {
                    echo " ***** Building Docker Image ***** "
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd"
                    sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                    echo " ***** PUSHING THE IMAGE TO DOCKER REPO ***** "
                    sh "docker push ${DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"

                }
            }
        }       
    }
}
