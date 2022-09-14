pipeline {
    agent any

    tools {
        jdk 'JDK 11'
        maven 'Maven_3.8.5'
    }

    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '3'))
    }

    parameters {
        string(name: 'JAIN_SLEE_SMPP_MAJOR_VERSION', defaultValue: '7.1.0', description: 'The major version for JAIN SLEE SMPP Community edition')
        string(name: 'CH_SMPP_VERSION', defaultValue: '6.1.0-5', description: 'The major version for cloudhopper smpp')
        string(name: 'SMPP_EXTENION_VERSION', defaultValue: '7.1.0-191', description: 'The major version for Extended SMPP Extension')
    }

    stages {
        stage('Set Version') {
            steps {
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                sh "mvn versions:commit"
            }
        }

        stage ('Build') {
            steps {
                script {
                    currentBuild.displayName = "#${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                    currentBuild.description = "Community JAIN SLEE SMPP"
                }
                sh "mvn -Dch.smpp.version=${params.CH_SMPP_VERSION} -Dsmpp-extensions.version=${params.SMPP_EXTENION_VERSION} clean install -DskipTests"
            }
        }

        stage ('Release') {
            steps {
                sh 'find . -type d -name \\"target\\" -exec r -r {} +'

                sh """mvn -Dch.smpp.version=${params.CH_SMPP_VERSION} \
                -Dsmpp-extensions.version=${params.SMPP_EXTENION_VERSION} clean install \
                -DskipTests -Pall -Prelease-wildfly -Drelease.dir=../../../${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"""
                sh 'rm -rf generated-docs'
                // compress the release wildfly
                sh "zip -r ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}.zip ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                // copy the docs folder to the root ang zip it
                sh 'mv resources/smpp/smpp-server-ra-docs/smpp-server-ra-docs-sources-asciidoc/target/generated-docs/ .'
                sh "zip -r smpp-ra-generated-docs.zip generated-docs"
                // remove the folder and re-create it again
                sh "rm -rf ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                sh "mkdir ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                sh "mv ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}.zip ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                sh "mv smpp-ra-generated-docs.zip ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
            }
        }

        stage ('Save Artifact') {
            steps {
                archiveArtifacts artifacts: "${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/", followSymlinks: false, onlyIfSuccessful: true
            }
        }

        stage('Push to Repo') {
            when { branch 'master' }
            steps {
                /*sshagent(credentials: ['4e708f2a-8b37-414f-a6d4-787690b87738']) {
                    sh "scp -r ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/ root@127.0.0.1:/var/www/html/NAIKERI/jain_slee_smpp/${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/"
                }*/
                sh "mkdir -p /var/www/html/NAIKERI/jain_slee_smpp/${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/"
                sh "cp ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/ /var/www/html/NAIKERI/jain_slee_smpp/${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}/"
                sh "rm -rf ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                sh "rm -f ${params.JAIN_SLEE_SMPP_MAJOR_VERSION}-${BUILD_NUMBER}.zip"
            }
        }
        stage ('Push to jFrog') {
            when{anyOf{branch 'master'; branch 'release'}}
		    steps{
                sh "mvn -Dch.smpp.version=${params.CH_SMPP_VERSION} -Dsmpp-extensions.version=${params.SMPP_EXTENION_VERSION} deploy"
		    }
        }
    }
}
