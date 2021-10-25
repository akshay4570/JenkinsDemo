#!groovy
import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def TEST_LEVEL = 'RunLocalTests'
    def FORCE_APP = 'force-app'
    def PATH = 'C:\\results.csv'
    def FORMAT = 'csv'

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    
    def toolbelt = tool 'toolbelt'
    def scanner = tool 'scanner'
   
    stage('checkout source') {
	// when running in multi-branch job temp
	    checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize DevHub'){
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'Salesforce Dev hub org authorization failed' }
        }

        stage('Static Code Analysis'){
            rc = command "${toolbelt} ${scanner}:run --target ${FORCE_APP} --outputfile ${PATH} --format ${FORMAT}"
        }
        stage('Run Tests on Data'){
            rc =  bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy -m ApexClass -l RunAllTestsInOrg -c -u ${HUB_ORG}"
        }

        stage('Deploy Code') {
        

            println rc
            
            // need to pull out assigned username
            if (isUnix()) {
            rmsg = sh returnStdout: true, script: "${toolbelt} force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
            }else{
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
            }
            
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
        }
    }
}
