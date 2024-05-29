pipeline {
    agent any
    tools {maven "maven"}
    environment {
        RHT_OCP4_DEV_USER = 'xudlbs'
        DEPLOYMENT_STAGE = 'shopping-cart-stage'
        DEPLOYMENT_PRODUCTION = 'shopping-cart-production'
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
        stage('Build Image') {
                environment { QUAY = credentials('QUAY_USER') }
                steps {
                    sh '''
                      mvn quarkus:add-extension \
                      -Dextensions="container-image-jib"
                      mvn quarkus:add-extension \
                      -Dextensions="kubernetes"
                      '''
                      sh '''
                        mvn package -DskipTests \
                        -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                        -Dquarkus.container-image.build=true \
                        -Dquarkus.container-image.registry=quay.io \
                        -Dquarkus.container-image.group=$QUAY_USR \
                        -Dquarkus.container-image.name=do400-deploying-environments \
                        -Dquarkus.container-image.username=$QUAY_USR \
                        -Dquarkus.container-image.password="$QUAY_PSW" \
                        -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                        -Dquarkus.container-image.push=true
                        '''

                }
            }
        
        stage('Deploy - Stage') {
            environment {
                APP_NAMESPACE = "xudlbs-shopping-cart-stage"
                QUAY = credentials('QUAY_USER')
            }
            steps {
                sh """
                  oc set image \
                  deployment ${DEPLOYMENT_STAGE} \
                  shopping-cart-stage=quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER} \
                  -n ${APP_NAMESPACE} 
                  """
            }
        } 
        stage('Deploy - Prod') {
            environment {
                APP_NAMESPACE = "xudlbs-shopping-cart-production"
                QUAY = credentials('QUAY_USER')
            }
            input { message 'Deploy to PROD?'}
            steps {
                sh '''
                oc set image \
                deployment ${DEPLOYMENT_PRODUCTION} \
                shopping-cart-production=quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER} \
                -n ${APP_NAMESPACE} --record
                '''
            }
        }
        }
    }
