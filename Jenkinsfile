library(
    identifier: 'jenkins-lib-common@v4.4.0',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        credentialsId: 'jenkins-integration-with-github-account',
        remote: 'git@github.com:zextras/jenkins-lib-common.git',
    ])
)

properties(defaultPipelineProperties())

pipeline {
    agent {
        node {
            label 'zextras-v1'
        }
    }
    environment {
        JAVA_OPTS = '-Dfile.encoding=UTF8'
        LC_ALL = 'C.UTF-8'
        jenkins_build = 'true'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '25'))
        timeout(time: 2, unit: 'HOURS')
        skipDefaultCheckout()
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    gitMetadata()
                    semanticRelease.guard()
                }
                container('jdk-21') {
                    sh 'apt-get update -qq && apt-get install -y ant -qq'
                }
            }
        }
        stage('Security Scan') {
            steps {
                gitleaksStage()
            }
        }
        stage('Bump version') {
            when {
                expression { env.GIT_IS_DEFAULT_BRANCH == 'true' }
            }
            steps {
                script {
                    semanticRelease()
                }
            }
        }
        stage('Build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-jenkins-gradle-properties-splitted', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    container('jdk-21') {
                        sh 'ant -propertyfile build.properties -Dartifactory_user="${ARTIFACTORY_USER}" -Dartifactory_password="${ARTIFACTORY_PASSWORD}" jar'
                    }
                }
            }
        }
        stage('Publish to maven') {
            when {
                buildingTag()
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'artifactory-jenkins-gradle-properties-splitted', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    container('jdk-21') {
                        sh 'ant -propertyfile build.properties -Dartifactory_user="${ARTIFACTORY_USER}" -Dartifactory_password="${ARTIFACTORY_PASSWORD}" publish-maven-all'
                    }
                }
            }
        }
    }
}
