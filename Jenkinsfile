import org.codehaus.mojo:build-helper-maven-plugin:3.6.0:parse-version

pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
        stages {
          stage('increment version') {
            steps {
                script {
                    echo "Incrementing app version"
                    sh 'mvn builder-helper:parse-version versions:set \
                       -DnewVersion=\\\$parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                       versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage("build jar") {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'

                }
            }
        }

        stage("build image") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t irschad/java-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push irschad/java-app:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage("deploy") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                    echo 'deploying the application...'
                }
            }
        }               
    }
} 
