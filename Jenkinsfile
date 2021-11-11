#!groovy
import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = '62f79840-fcf5-44c6-9a12-12ab5bd0e93a'
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def TEST_LEVEL = 'RunLocalTests'
    def FORCE_APP = "force-app"
    def PATH = 'C:\\results.csv'
    def FORMAT = 'csv'
    def CONVERT = 'convert'

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    
    //def sfdx = tool 'sfdx'
   
    stage('checkout source') {
	// when running in multi-branch job temp
	    checkout scm
    }

    stage('Installations and Dependencies') {
        bat 'npm install -i sfdx'
        bat 'sfdx plugins:install @salesforce/sfdx-scanner'
    }
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize DevHub'){
            if (isUnix()) {
                rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'Salesforce Dev hub org authorization failed' }
        }
        // stage('Fetch Delta Changes'){
        //     rc = bat returnStdout: true, script:  """
        //                                              git config remote.origin.fetch \"+refs/heads/*:refs/remotes/origin/*\" 
        //                                              git fetch --all 
        //                                              git checkout pr 
        //                                              git --no-pager diff --name-status pr origin/QA 
        //                                              \"${sfdx}\" sgd:source:delta --to pr --from origin/QA_Release1 --repo . --output .
        //                                              cat package/package.xml
        //                                           """ 
        // }
        stage('Static Code Analysis'){
            rc = bat returnStdout: true, script:  "sfdx scanner:run --target force-app --format csv"
        }
        stage('Convert to Data'){
            rc =  bat returnStdout: true, script: "sfdx force:source:convert --rootdir=force-app --outputdir=convert"
        }

        stage('Deploy Code') {
        

            println rc
            
            // need to pull out assigned username
            if (isUnix()) {
            rmsg = sh returnStdout: true, script: "sfdx force:mdapi:deploy --deploydir=convert --testlevel=RunLocalTests --checkonly -u ${HUB_ORG}"
            }else{
            rmsg = bat returnStdout: true, script: "sfdx force:mdapi:deploy --deploydir=convert --testlevel=RunLocalTests --checkonly -u ${HUB_ORG}"
            }
            
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
        }
    }
}
