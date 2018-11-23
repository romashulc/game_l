#!/usr/bin/env groovy

timestamps {
    node ('agent1') {
        stage ('new - Checkout') {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/romashulc/game_l.git']]])
            GIT_TAG_NAME = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
            return GIT_TAG_NAME
        }
        stage ('new - Build') {
            withMaven(maven: 'maven') {
                sh 'mvn clean install'
            }
            fileOperations([fileCopyOperation(excludes: '', flattenFiles: true, includes: 'gameoflife-web/target/gameoflife.war', targetLocation: 'v3.2/')])
            fileOperations([fileZipOperation(folderPath: 'v3.2/')])
            junit 'gameoflife-web/target/surefire-reports/**.xml'
            googleStorageUpload([bucket: 'gs://my_pro', credentialsId: 'new', pattern: 'v3.2.zip'])
        }
        stage ('deploy - Build') {
            googleStorageDownload([bucketUri: 'gs://my_pro/v3.2.zip', credentialsId: 'new', localDirectory: ''])
            fileOperations([fileUnZipOperation(filePath: 'v3.2.zip', targetLocation: '')])
            sshPublisher(publishers: [sshPublisherDesc(configName: 'deploy_server', transfers: [sshTransfer(execCommand: '', execTimeout: 120000, sourceFiles: 'v3.2/gameoflife.war', removePrefix: 'v3.2')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
        }
    }
}