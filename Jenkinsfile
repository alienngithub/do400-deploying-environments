pipeline {
    agent any
    tools {maven "maven"}
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
        stage('Build Image') {
                environment { QUAY = credentials('QUAY_USER') }
                steps {
                    sh '''
                      mvn quarkus:add-extension \
                      -Dextensions="container-image-jib"
                      '''
                      sh '''
                        mvn package -DskipTests \
                        -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                        -Dquarkuscontainer-image.build=true \
                        -Dquarkuscontainer-image.registry=quay.io \
                        -Dquarkuscontainer-image.group=$QUAY_USR \
                        -Dquarkuscontainer-image.name=do400-deploying-environments \
                        -Dquarkuscontainer-image.username=$QUAY_USR \
                        -Dquarkuscontainer-image.password="$QUAY_PSW" \
                        -Dquarkuscontainer-image.push=true
                        '''

                }
            }
        }
    }
