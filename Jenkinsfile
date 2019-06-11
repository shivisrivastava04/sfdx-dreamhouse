#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests\${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def HUB_KEY=env.HUB_KEY_FILE_PATH_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def SFDX_HOME = "C:\\Program Files\\Salesforce CLI\\bin\\"
    def SFDX_USE_GENERIC_UNIX_KEYCHAIN = true
    def secretKey="C:\\openssl\\bin\\server.key"

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println SFDX_HOME
    println CONNECTED_APP_CONSUMER_KEY
    //def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

	stage('Create Scratch Org') {

        rc = bat returnStatus: true, script: "\"${SFDX_HOME}sfdx\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${secretKey} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
        if (rc != 0) { error 'hub org authorization failed' }

        // need to pull out assigned username 
        rmsg = bat returnStdout: true, script: "\"${SFDX_HOME}sfdx\" force:org:create -f config\workspace-scratch-def.json --setalias test --durationdays 7 --setdefaultusername"
        printf rmsg
        def jsonSlurper = new JsonSlurperClassic()
        def robj = jsonSlurper.parseText(rmsg)
        if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
        SFDC_USERNAME=robj.username
        robj = null
        
    }
	stage('Push To Test Org') {
        rc = bat returnStatus: true, script: "\"${SFDX_HOME}sfdx\" force:src:push --all --username ${SFDC_USERNAME}"
        if (rc != 0) {
            error 'push all failed'
        }
        // assign permset
        rc = bat returnStatus: true, script: "\"${SFDX_HOME}sfdx\" force:permset:assign --username ${SFDC_USERNAME} --name DreamHouse"
        if (rc != 0) {
            error 'push all failed'
        }
	}
	 stage('Run Apex Test') {
        bat "mkdir ${RUN_ARTIFACT_DIR}"
        timeout(time: 120, unit: 'SECONDS') {
            rc = bat returnStatus: true, script: "\"${SFDX_HOME}sfdx\" force:apex:test:run --testlevel RunLocalTests --username ${HUB_ORG}"
            if (rc != 0) {
                error 'apex test run failed'
            }
        }
    }

    stage('collect results') {
        junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
    }
}
