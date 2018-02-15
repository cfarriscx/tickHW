#!groovy
import hudson.model.*

// define branch before node so it is a global variable available in all stages.


try {
    def branch = ''
    node {
        node('skopeo-jenkins-slave') {
            stage ('move-image') {
                skopeo copy docker-registry.default.svc:5000/twitter-cicd/cfarriscx-master docker-registry.default.svc:5000/twitter-cicd/cfarriscx-testbranch
            }
        }
        stage('checkout-and-test') {
            openshiftImageStream apiURL: '', authToken: '', name: 'cfarriscx-master', namespace: 'twitter-cicd', tag: 'latest', verbose: 'true'
            sh 'docker -help'
            // Read payload which is a submitted JSON request from github and write to temp file
            sh 'echo "$payload" > tempGitFile.json'
            // From the temp file place into variable
            def fromgithook = readJSON file: 'tempGitFile.json'
            // find branch name and set to lower case for environment variables
            branch = fromgithook.ref
            def isDeleted = fromgithook.deleted
            println isDeleted
            def branchFull = branch
            def lowercaseBranch = branch.toLowerCase();
            def user = fromgithook.pusher.name
            branch = branch.substring(branch.lastIndexOf("/") + 1)
            branch = branch.toLowerCase()
            

            sh 'oc project twitter-cicd'
            // Check for new branch and existing openshift buildconfig
            
            sh """oc get dc -l BRANCH=$branch &> tempGetDC"""
            def existingDeploymentConfig = readFile('tempGetDC').trim()
            println existingDeploymentConfig
            // Check git message for deleted branch. If deleted then clean resources
            if(isDeleted == 'true') {
                // delete all with label
                """oc delete all -l BRANCH=$branch"""
                """oc delete pvc -l BRANCH=$branch"""
                """oc delete secret -l BRANCH=$branch"""
            } else if(existingDeploymentConfig == "No resources found.") {
                // new branch so generate DC from template
                sh """oc process nodejs-mongo-jenkinspipe \
                -p NAME=$user-$branch \
                -p SOURCE_REPOSITORY_URL=https://github.com/cfarriscx/tickHW.git \
                -p SOURCE_REPOSITORY_REF=$branchFull \
                -p DATABASE_NAME=$branch \
                -p DATABASE_SERVICE_NAME=$branch-mongodb \
                -l BRANCH=$branch \
                | oc create -f -"""
            } else {
                // old branch with existing DC so launch build and deploy
                openshiftBuild apiURL: '', authToken: '', bldCfg: """$user-$branch""", buildName: '', checkForTriggeredDeployments: 'true', commitID: '', namespace: '', showBuildLogs: 'true', verbose: 'false', waitTime: '', waitUnit: 'sec'
                openshiftDeploy depCfg: """$user-$branch""", verbose: 'false'
                openshiftVerifyDeployment depCfg: """$user-$branch""", verbose: 'false'
            }

        }

        stage('Approve Clean and Delete') {
            timeout(time: 2, unit: 'DAYS') {
                input message: 'Delete app based on branch'
            }
        }

        stage('Clean and Delete') {
            sh """oc delete all -l BRANCH=$branch"""
            sh """oc delete pvc -l BRANCH=$branch"""
            sh """oc delete secret -l BRANCH=$branch"""
        }
    }
} catch (err) {
    echo "in catch block"
    echo "Caught: ${err}"
    currentBuild.result = 'FAILURE'
    throw err
}
/*

try {
    node {
        stage('Build') {
            openshiftBuild apiURL: '', authToken: '', bldCfg: 'simple-nodejs-dev', buildName: '', checkForTriggeredDeployments: 'true', commitID: '', namespace: '', showBuildLogs: 'true', verbose: 'false', waitTime: '', waitUnit: 'sec'
            openshiftVerifyBuild bldCfg: 'simple-nodejs-dev', checkForTriggeredDeployments: 'true', showBuildLogs: 'true', verbose: 'false'

            openshiftTag alias: 'false', destStream: 'simple-nodejs-dev', destTag: 'dev', srcStream: 'simple-nodejs-dev', srcTag: 'latest', verbose: 'false'
        }
        stage('Deploy to Dev') {
            openshiftDeploy depCfg: 'simple-nodejs-dev', verbose: 'false'
            openshiftVerifyDeployment depCfg: 'simple-nodejs-dev', verbose: 'false'
            
            openshiftTag alias: 'false', destStream: 'simple-nodejs-dev', destTag: 'qa', srcStream: 'simple-nodejs-dev', srcTag: 'dev', verbose: 'false'
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

            openshiftTag alias: 'false', destStream: 'simple-nodejs-dev', destTag: 'prod', srcStream: 'simple-nodejs-dev', srcTag: 'qa', verbose: 'false'
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

*/