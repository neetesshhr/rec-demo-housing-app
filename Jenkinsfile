#!groovy

node {

    // def SF_CONSUMER_KEY=env.sfdxkey
    def SF_USERNAME=env.DEV_HUB_USER
   
    def DEPLOYDIR='src'
    def TEST_LEVEL='RunLocalTests'
    def SF_INSTANCE_URL = env.DEV_HUB_URL ?: "https://login.salesforce.com"


  

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
			sh "sfdx auth:jwt:grant -r ${SF_INSTANCE_URL} -i ${consumer_key} -f ${server_key_file} --username ${SF_USERNAME} -a UAT -d"
	
		}


		// -------------------------------------------------------------------------
		// Deploy metadata and execute unit tests.
		// ---------------------------------------q----------------------------------

        stage('Delete Scratch Org if present') {
        sh '''
            orgList=$(sfdx force:org:list --clean --json)
            scratchOrg=$(echo $orgList | jq '.result.scratchOrgs[] | select(.alias == "rec-house")')
            if [ ! -z "$scratchOrg" ]
            then
                sfdx force:org:delete -u $scratchOrg.username -p
            fi
        '''
    }
    stage('Create Scratch Org') {
        sh 'sfdx force:org:create -s -f config/project-scratch-def.json -a rec-house'
    }

     stage('source push'){
        sh "sfdx force:source:push"
     }
		// -------------------------------------------------------------------------
		// Example shows how to run a check-only deploy. in the org
		// -------------------------------------------------------------------------
        stage('Retrive the metadatafrom the org'){
            sh "sfdx force:source:retrieve -x manifest/package.xml -r src2 -u  rec-house"
        }

         stage('Deploy the metadata') {
        // Run all tests in the org and check code coverage
        def testResult = sh(script: "sfdx force:source:deploy -p src2 --testlevel RunAllTestsInOrg --runtests all --json -u rec-house", returnStdout: true)
        def testJson = readJSON(text: testResult)

        // Check if code coverage is less than 100%
        def coverage = testJson.result.details.runTestResult.coverage
        if (coverage != null && coverage.aggregateCoverage < 100) {
            // If code coverage is less than 100%, log a warning and continue the pipeline
            echo "Code coverage is less than 100% (${coverage.aggregateCoverage}%). Skipping deployment."
            // sh 'sfdx force:org:open'
        } else {
            // Deploy metadata if code coverage is 100%
            sh 'sfdx force:deploy --wait 10 --json --loglevel error --ignoreerrors'
        }
    }
	    }
	}
}


