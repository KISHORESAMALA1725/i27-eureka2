pipeline {
    agent {
        label "k8s-slave"
    }

    tools {
        maven "maven-3.8.8"
        jdk "JDK-17"
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
    }
}
