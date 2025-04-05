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
    }
}
