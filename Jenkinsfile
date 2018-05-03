#!groovy

import hudson.AbortException

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

/**
 *	Copyright (c) 2017 SAP SE or an SAP affiliate company.  All rights reserved.
 *
 *	This software is the confidential and proprietary information of SAP
 *	("Confidential Information"). You shall not disclose such Confidential
 *	Information and shall use it only in accordance with the terms of the
 *	license agreement you entered into with SAP.
*/

@Library('piper-library-os@a8a9281093915734957a2589d08fa826f3102b61') _

def CONFIG_FILE = '.pipeline/config.yml'

//
// CM_CLIENT RELATED PROPERTIES START

  // Build server specific, should not be part of the project config
  def CM_CLIENT_EXECUTABLE='$JENKINS_HOME/userContent/cmclient/v0.0.2-SNAPSHOT/bin/cmclient'

  // Company specific, should be part of a configuration specific config layer
  // (which we do not have yet).
  def CM_ENDPOINT='https://fbt.wdf.sap.corp:44300/sap/opu/odata/SAP/AI_CRM_GW_CM_CI_SRV/'

  // The following properties are project specific
  def CM_CREDENTIALS_KEY='CM'

  // Should be provided e.g. via git commit message.
  def CM_CHANGE_ID='8000050359'

// CM_CLIENT RELATED PROPERTIES END
//

node() {
  //Global variables:
  APP_PATH = 'src'
  SRC = "${WORKSPACE}/${APP_PATH}"

  stage("Clone sources and setup environment"){
    deleteDir()
    dir(APP_PATH) {
      checkout scm
      setupCommonPipelineEnvironment script: this, configFile: CONFIG_FILE
      prepareDefaultValues script: this
    }

    proxy = commonPipelineEnvironment.getConfigProperty('proxy') ?: ''
    httpsProxy = commonPipelineEnvironment.getConfigProperty('httpsProxy') ?: ''
  }

  stage("Check Change Document") {
    def rc = -1
    withCredentials([usernamePassword(
                       credentialsId: CM_CREDENTIALS_KEY,
                       passwordVariable: 'CM_PASSWORD',
                       usernameVariable: 'CM_USER')]) {

          rc = sh returnStatus: true,
                  script: """${CM_CLIENT_EXECUTABLE} -t SOLMAN \
                                                     -e ${CM_ENDPOINT} \
                                                     -u ${CM_USER} \
                                                     -p '${CM_PASSWORD}' \
                                                 is-change-in-development \
                                                     --return-code \
                                                     -cID ${CM_CHANGE_ID}"""
    }

    if(rc == 3) { // change is not in status "in development"
      throw new AbortException("Change '${CM_CHANGE_ID}' is not in status 'in development'.")
    } else if (rc != 0) { // values != 0 indicates a failure
      throw new AbortException("Error while retrieving status for change '${CM_CHANGE_ID}'.")
    }

    echo "Change '${CM_CHANGE_ID}' is in required status 'in development'."
  }

  stage("Build Fiori App"){
    dir(SRC){
      withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) {
        MTAR_FILE_PATH = mtaBuild script: this, buildTarget: 'NEO'
      }
    }
  }

  stage('Upload Into Transport') {

    withCredentials([usernamePassword(
                       credentialsId: CM_CREDENTIALS_KEY,
                       passwordVariable: 'CM_PASSWORD',
                       usernameVariable: 'CM_USER')]) {

        sh script: """#!/bin/bash
                      tID=`${CM_CLIENT_EXECUTABLE} -t SOLMAN \
                                                  -e ${CM_ENDPOINT} \
                                                  -u ${CM_USER} \
                                                  -p ${CM_PASSWORD} \
                                              create-transport \
                                                  -cID ${CM_CHANGE_ID}`
                      if [ $? == 0 ]
                      then
                          echo "Transport request '\${tID}' created."
                      else
                          echo "Cannot create transport."
                          exit 1
                      fi

"""
      }
  }
  stage("Deploy Fiori App"){
    dir(SRC){
      withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) {
        neoDeploy script: this, archivePath: MTAR_FILE_PATH
      }
    }
  }

  stage("Smoke Test") {
          def APPLICATION_HOST='helloworld-d025390trial.dispatcher.hanatrial.ondemand.com'
          def APPLICATION_URL="https://${APPLICATION_HOST}?hc_reset"
    sh """#!/bin/bash
          curl \
            --fail \
            --noproxy '${APPLICATION_HOST}' \
            --verbose \
            --insecure \
            '${APPLICATION_URL}' """
  }
}

