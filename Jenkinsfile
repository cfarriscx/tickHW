#!groovy
import hudson.model.*


try {
    node {
        state('Build') {
            checkout scm
            sh 'npm install'
            sh 'npm test'
            openshiftBuild apiURL: '', authToken: '', bldCfg: 'simple-nodejs-dev', buildName: '', checkForTriggeredDeployments: 'true', commitID: '', namespace: '', showBuildLogs: 'true', verbose: 'false', waitTime: '', waitUnit: 'sec'
            openshiftVerifyBuild bldCfg: 'simple-nodejs-dev', checkForTriggeredDeployments: 'true', showBuildLogs: 'true', verbose: 'false'

        }
        stage('Deploy to Dev') {
            openshiftDeploy depCfg: 'simple-nodejs-dev', verbose: 'false'
            openshiftVerifyDeployment depCfg: 'simple-nodejs-dev', verbose: 'false'
        }
        stage('Approve QA Deployment') {
            timeout(time: 2, unit: 'DAYS') {
                input message: 'Do you want to deploy into Q&A?'
            }
        }
        // Publish to a QA environment
        stage('Deploy to QA') {
            openshiftDeploy depCfg: 'simple-nodejs-qa', verbose: 'false'
            openshiftVerifyDeployment depCfg: 'simple-nodejs-qa', verbose: 'false'
        }
        // Wait until authorization to push to production
        stage('Approve Production Deployment') {
            timeout(time: 2, unit: 'DAYS') {
                input message: 'Do you want to deploy into production?'
            }
        }
        // Push to production
        stage('Deploy to Production') {
            openshiftDeploy depCfg: 'simple-nodejs-dev', verbose: 'false'
            openshiftVerifyDeployment depCfg: 'simple-nodejs-dev', verbose: 'false'
        } 
    }
} catch (err) {
    echo "in catch block"
    echo "Caught: ${err}"
    currentBuild.result = 'FAILURE'
    throw err
}