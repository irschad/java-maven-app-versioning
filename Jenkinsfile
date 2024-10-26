def gv

pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                    echo "Executing pipeline for branch $BRANCH_NAME"
                }
            }
        }
        stage("build jar") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                    gv.buildJar()

                }
            }
        }

        stage("build image") {
            steps {
                script {
                    gv.buildImage()
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
                    gv.deployApp()
                }
            }
        }               
    }
} 
