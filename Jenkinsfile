#!groovy

node {

    // def SF_CONSUMER_KEY=env.sfdxkey
    def SF_USERNAME=env.DEV_HUB_USER
    // def SERVER_KEY_CREDENTIALS_ID=env.clientid
    def DEPLOYDIR='src'
    def TEST_LEVEL='RunLocalTests'
    def SF_INSTANCE_URL = env.DEV_HUB_URL ?: "https://login.salesforce.com"


    // def toolbelt = tool 'toolbelt'
    
    // println 'keyname'	
    // println SF_USERNAME
    // println SF_CONSUMER_KEY
    // println SERVER_KEY_CREDENTIALS_ID

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) {	
	
    withCredentials([file(credentialsId: 'sfdxkey', variable: 'server_key_file'), string(credentialsId: 'clientid', variable: 'consumer_key')]) {
    // some block

	    
		// -------------------------------------------------------------------------
		// Authenticate to Salesforce using the server key.
		// -------------------------------------------------------------------------

		stage('Authorize to Salesforce') {
			sh "sfdx auth:jwt:grant -r ${SF_INSTANCE_URL} -i ${consumer_key} -f ${server_key_file} --username ${SF_USERNAME} -a UAT"
	
		}


		// -------------------------------------------------------------------------
		// Deploy metadata and execute unit tests.
		// -------------------------------------------------------------------------

		stage('Deploy and Run Tests') {
		    sh "sfdx force:source:deploy --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"

		}


		// -------------------------------------------------------------------------
		// Example shows how to run a check-only deploy.
		// -------------------------------------------------------------------------

		stage('Check Only Deploy') {
		   sh "sfdx force:source:deploy --checkonly --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"

		}
	    }
	}
}


