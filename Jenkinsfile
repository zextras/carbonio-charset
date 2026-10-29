library(
    identifier: 'jenkins-lib-common@v5.1.1',
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
                container('jdk-21') {
                    sh 'ant -propertyfile build.properties jar'
                }
            }
        }
        stage('Publish to maven') {
            when {
                buildingTag()
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-prod-jenkins-ci-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                    container('jdk-21') {
                        sh 'ant -propertyfile build.properties -Dnexus_user="${NEXUS_USER}" -Dnexus_password="${NEXUS_PASSWORD}" publish-maven-all'
                    }
                }
            }
        }
    }
}
