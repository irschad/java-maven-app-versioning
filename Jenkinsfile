
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
                    sh 'mvn build-helper:parse-version versions:set \
                       -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                       versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
          }

        stage('build jar') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'

                }
            }
        }

        stage('build image') {
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

        stage('deploy') {
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
        stage('commit version update') {
              when {
                  expression {
                      BRANCH_NAME == "master"
                  }
              }
              steps {
                  script {
                      withCredentials([usernamePassword(credentialsId: 'jenkinspush', passwordVariable: 'PAT' , usernameVariable: 'USER')]) {
                       //passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                       //   sh 'git checkout master'
                          sh 'git remote set-head origin master'
                          sh 'git config --global user.email "jenkins@example.com"'
                          sh 'git config --global user.name "jenkins"'
                          sh 'git status'
                          sh 'git add .'
                          sh 'git branch'
                          sh 'git config --list'
                          sh "git remote set-url origin https://${PAT}@github.com/irschad/java-maven-app-versioning.git"
                          sh "git commit -m 'ci: version bump'"
                          sh 'git push origin HEAD:master'
                    }
                  }
              }
          }

    }
}
