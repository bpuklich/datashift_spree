#!/usr/bin/env groovy

node {
    try {
        stage('Checkout') {
            checkout scm
        }
        stage('Build') {
            sh '''#!/bin/bash -l
                set -e
                cd spec
                rvm use ruby-2.2.5@datashift_spree-$BRANCH_NAME
                gem install bundler --no-document
                bundle install
            '''
        }
        stage('Test') {
            sh '''#!/bin/bash -l
                set -e
                cd spec
                rvm use ruby-2.2.5@datashift_spree-$BRANCH_NAME
                bundle exec rspec -c -I . .
            '''
        }
        stage('Cleanup') {
            echo 'Cleaning workspace (if successful)'
            sh '''#!/bin/bash -l
                rvm --force gemset delete datashift_spree-$BRANCH_NAME
            '''
            step([$class: 'WsCleanup', deleteDirs: true, patterns: [[pattern: '.git/**', type: 'EXCLUDE']]])
        }
    } catch (e) {
        currentBuild.result = "FAILED"
        throw e
    } finally {
        def buildStatus = currentBuild.result ? currentBuild.result : 'SUCCESSFUL'
        def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def details = "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - ${buildStatus}:\n" +
                "\n" +
                "Check console output at $BUILD_URL to view the results."
        emailext recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                subject: subject,
                body: details
    }
}
