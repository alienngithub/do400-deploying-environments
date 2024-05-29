pipeline {
    agent {
        node {
            label 'nodejs'
        }
    }
    stages {
        stage('Tests') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Package') {
            steps {
                sh '''
                mvn package -DskipTests \
                -Dquarkus.package.type=uber-jar
                '''
                archiveArtifacts 'target/*.jar'

            }
        }
    }
}