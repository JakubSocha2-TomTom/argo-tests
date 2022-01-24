@Library('jenkins-pipeline-library@2.2.1') _

pipeline {
    agent {
        node {
            label 'jenkins-slave-ci-2GB'
        }
    }

    triggers {
        bitbucketPush()
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '20', artifactNumToKeepStr: '10', daysToKeepStr: '20', numToKeepStr: '10')
        timestamps()
        skipDefaultCheckout(true)
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    cleanWs()
                    checkout scm
                }
            }
        }
        stage('Build and Unit Test') {
            steps {
                sh script: 'bash ./gradlew clean build --no-daemon', label: 'gradlew clean build'
            }
        }
        stage('Release') {
            when {
                branch 'feature/NOJIRA-axion_test'
            }
            stages {
                stage('Create release tag') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'github_osm_ci', usernameVariable: 'GITHUB_APP', passwordVariable: 'GITHUB_ACCESS_TOKEN')]) {
                                sh './gradlew release -x test --info -Prelease.disableChecks -Prelease.pushTagsOnly -Prelease.customUsername=$GITHUB_APP -Prelease.customPassword=$GITHUB_ACCESS_TOKEN'
                            }
                        }
                    }
                }
                stage('Publish to Nexus') {
                    steps {
                        script {
                            try {
                                sh './gradlew publish -x test --info'
                            } catch (err) {
                                echo '[ERROR] Skipping publishing to nexus repository due to an error'
                                echo err.toString()
                            }
                        }
                    }
                }
            }
        }
    }
}