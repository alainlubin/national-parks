pipeline {
    agent any
    environment {
        HAB_NOCOLORING = true
        HAB_BLDR_URL = 'https://bldr.habitat.sh/'
        HAB_ORIGIN = 'nrycar'
    }
    stages {
        stage('scm') {
            steps {
                git url: 'https://github.com/ChefRycar/national-parks.git', branch: 'master'
            }
        }
        stage('build') {
            steps {
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                    script {
                       sh 'hab origin key download --secret $HAB_ORIGIN -z $HAB_AUTH_TOKEN'
                       sh 'hab origin key download $HAB_ORIGIN'
                    }
                }
                habitat task: 'build', directory: '.', origin: "${env.HAB_ORIGIN}", docker: true
            }
        }
        stage('upload') {
            steps {
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                    habitat task: 'upload', authToken: env.HAB_AUTH_TOKEN, lastBuildFile: "${workspace}/results/last_build.env", bldrUrl: "${env.HAB_BLDR_URL}"
                }
            }
        }
        stage('promote-to-canary') {
            steps {
                script {
                    env.HAB_PKG = sh (
                        script: "ls -t "{$workspace}"/results | grep hart | head -n 1",
                        returnStdout: true
                        ).trim()
                }
                withCredentials([string(credentialsId: 'hab-depot-token', variable: 'HAB_AUTH_TOKEN')]) {
                  habitat task: 'promote', channel: "canary", authToken: "${env.HAB_AUTH_TOKEN}", artifact: "${env.HAB_PKG}", bldrUrl: "${env.HAB_BLDR_URL}"
                }
            }
        }
    }
}