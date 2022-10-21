pipeline {
    agent any
    environment {
        production_jwt_key_file = credentials('SFDX-PRODUCTION-KEY') 
        PROD_CONNECTED_APP_CONSUMER_KEY = credentials('sf-sfdx-app-consumer-key-production') 
        PROD_USER = credentials('sf-sfdx-user-production')  
        
    }
    stages {
        stage('checkout source') {
            steps {
                checkout scm
            }
        }
        stage('Create ScratchOrg') {
            steps {
                script {
                    if (isUnix()) {
                        //update Salesforce CLI
                        update = sh(script: "\"${env.SFDX_CLI}\" update")
                        //Login to Prod using connected app consumer key, user name, prod url and OpenSSL certificate and key
                        //test
                        login = sh(returnStatus: true, script: "\"${env.SFDX_CLI}\" auth:jwt:grant --clientid 3MVG9WtWSKUDG.x4A4I1E1o5ll5tjOK71TFl3t.UvNsF2btB6WTVvUfplndUVu9uHmVaQV4WfapwP8UNjJkV8 --username mafarouq@leyton.com --jwtkeyfile ${production_jwt_key_file} --loglevel DEBUG --setdefaultdevhubusername --instanceurl ${env.SFDX_PROD_URL} --setalias HubOrg")
                        //login = sh("test")
                        //Create scratchOrg from Prod
                        scratchOrg = sh(returnStatus: true, script: "\"${env.SFDX_CLI}\" force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1")
                        if (scratchOrg != 0) {
                            error 'Salesforce scratch org creation failed.'
                        }	
                    }else{
                        //update Salesforce CLI
                        update = bat(script: "\"${env.SFDX_CLI}\" update")
                        //Login to Prod using connected app consumer key, user name, prod url and OpenSSL certificate and key
                        login = bat(returnStatus: true, script: "\"${env.SFDX_CLI}\" auth:jwt:grant --clientid ${PROD_CONNECTED_APP_CONSUMER_KEY} --username ${PROD_USER} --jwtkeyfile ${production_jwt_key_file} --loglevel DEBUG --setdefaultdevhubusername --instanceurl ${env.SFDX_PROD_URL} --setalias HubOrg")
                        //Create scratchOrg from Prod
                        scratchOrg = bat(returnStatus: true, script: "\"${env.SFDX_CLI}\" force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1")
                        if (scratchOrg != 0) {
                            error 'Salesforce scratch org creation failed.'
                        }
                    }
                    if (login != 0) { 
                        error 'Hub Org authorization failed.' 
                    }
                    else{
                        println 'SFDX AUTHENTICATED !'
                    }                
                }
            }
        }
        stage('Deploy to ScratchOrg') {
            steps {
                script {
                        def filename = "${env.SFDX_CLASSES}"
                        def str = filename.split(",")
                        def classes = ""
                        for( String values : str ){
                                classes = classes + "ApexClass:" + values
                                if(values != str.last())
                                     classes = classes + ","
                        }
                        println classes
                    if (isUnix()) {
                        try{
                            //Push changes from Git Lab to scratch org
                            push = sh (returnStatus: true, script: "\"${env.SFDX_CLI}\" force:source:deploy -m "+classes+" --targetusername ciorg")
                        }catch(err){
                            //Delete Scratch org
                            logout = sh (returnStatus: true, script: "\"${env.SFDX_CLI}\" force:org:delete -p -u ciorg")
                            error 'Push to scratch org creation failed.'
                        }
                    }else{
                         try{
                            //Push changes from Git Lab to scratch org
                            push = bat (returnStatus: true, script:  "\"${env.SFDX_CLI}\" force:source:deploy -m "+classes+" --targetusername ciorg")
                        }catch(err){
                            //Delete Scratch org
                            logout = bat (returnStatus: true, script: "\"${env.SFDX_CLI}\" force:org:delete -p -u ciorg")
                            error 'Push to scratch org creation failed.'
                        } 
                    }
                }
            }
        }
        stage('Run Tests in ScratchOrg') {
            steps {
                script {
                    if (isUnix()) {
                        try{
                            //Run tests in scratch org
                            testres = sh (returnStdout: true, script:"\"${env.SFDX_CLI}\" force:apex:test:run --targetusername ciorg --wait 10 --classnames ${env.SFDX_TEST_CLASSES} -c -r human")
                        }catch(err){
                            //Delete Scratch org
                            logout = sh (returnStdout: true, script: "\"${env.SFDX_CLI}\" force:org:delete -p -u ciorg")
                            error 'Scratch org tests failed.'
                        }
                    }else{
                         try{
                            //Run tests in scratch org
                            testres = bat (returnStdout: true, script:"\"${env.SFDX_CLI}\" force:apex:test:run --targetusername ciorg --wait 10 --classnames ${env.SFDX_TEST_CLASSES} -c -r human")
                        }catch(err){
                            //Delete Scratch org
                            logout = bat (returnStdout: true, script: "\"${env.SFDX_CLI}\" force:org:delete -p -u ciorg")
                            error 'Scratch org tests failed.'
                        }
                    }
                }
            }
        }
       stage('Auth to Prod') {
            steps {
                script {
                    if (isUnix()) {
                        //Delete Scratch org
				        logout = sh (returnStatus: true, script: "${env.toolbelt} force:org:delete -p -u ciorg")
				        //Log out from Prod
				        logout = sh (returnStatus: true, script: "echo y | ${env.toolbelt} auth:logout --targetusername HubOrg")
				        //Log in to SandBox
				        slogin = sh (returnStatus: true, script: "${env.toolbelt} auth:jwt:grant --clientid ${PROD_CONNECTED_APP_CONSUMER_KEY} --username ${PROD_USER} --jwtkeyfile ${production_jwt_key_file} --setdefaultdevhubusername --instanceurl ${env.SFDX_PROD_URL} --setalias HubOrg")
                    }else{
                        //Delete Scratch org
				        logout = bat (returnStatus: true, script: "${env.toolbelt} force:org:delete -p -u ciorg")
				        //Log out from Prod
				        logout = bat (returnStatus: true, script: "echo y | ${env.toolbelt} auth:logout --targetusername HubOrg")
				        //Log in to SandBox
				        slogin = bat (returnStatus: true, script: "${env.toolbelt} auth:jwt:grant --clientid ${PROD_CONNECTED_APP_CONSUMER_KEY} --username ${PROD_USER} --jwtkeyfile ${production_jwt_key_file} --setdefaultdevhubusername --instanceurl ${env.SFDX_PROD_URL} --setalias HubOrg")
                    }
                }
            }
        }
        stage('Deploy to Prod') {
            steps {
                script {
			def deployResult
                    if (isUnix()) {
                        try{
                            //Deploy and check code coverage in SandBox
                            def filename = "${env.SFDX_CLASSES}"
                            def str = filename.split(",")
                            def classes = ""
                            for( String values : str ){
                                    classes = classes + "ApexClass:" + values
                                    if(values != str.last())
                                        classes = classes + ","
                            }
                            println classes
                            deployResult = sh (returnStdout: true, script: "\"${env.SFDX_CLI}\" force:source:deploy -m "+classes+" -u ${PROD_USER} -l RunSpecifiedTests -r \"${env.SFDX_TEST_CLASSES}\" --targetusername HubOrg")
                            //Log out from SnadBox
                            logout = sh (returnStatus: true, script: "echo y | \"${env.SFDX_CLI}\" auth:logout --targetusername HubOrg ")
                             println '********* ' + deployResult
			     println 'Deploy succeed.'
                        }catch(err){
                            //Show tests result if deploy fail
                            println testres
                            //Log out from SnadBox
                            logout = sh (returnStatus: true, script: "echo y |\"${env.SFDX_CLI}\" auth:logout --targetusername HubOrg ")
                            error 'Deploy failed.'
                        }
                    }else{
                        try{
                            //Deploy and check code coverage in SandBox
				println '1111111111111111111111111111111&&'
                           // deployResult = bat (returnStdout: true, script:  "\"${env.SFDX_CLI}\" force:source:deploy -m "+classes+" --targetusername HubOrg"/*"\"${env.SFDX_CLI}\" force:source:deploy -m "+classes+" -u ${PROD_USER} -l RunSpecifiedTests -r \"${env.SFDX_CLASSES}\" --targetusername HubOrg"*/)
                          logout = bat (returnStatus: true, script: "echo y | \"${env.SFDX_CLI}\" auth:logout --targetusername HubOrg ")
				println '2222222222222222222222222222222222222'
				//Log out from SnadBox
                           // logout = bat (returnStatus: true, script: "echo y | \"${env.SFDX_CLI}\" auth:logout --targetusername HubOrg ")
				println '3333333333333333333333333333'
			    println '********* ' + deployResult
                            println 'Deploy succeed.'
                        }catch(err){
                            //Show tests result if deploy fail
				println '********* ' + deployResult
                            println testres
                            //Log out from SnadBox
                            logout = bat (returnStatus: true, script: "echo y | \"${env.SFDX_CLI}\" auth:logout --targetusername HubOrg ")
                            error 'Deploy failed.'
                        }
                    }
                }
            }
        }
    
   
    }
}
