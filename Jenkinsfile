#!groovy

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

@Library('piper-library-os') _

def CONFIG_FILE = '.pipeline/config.yml'

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

  stage("Build Fiori App"){
    dir(SRC){
      withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) {
        MTAR_FILE_PATH = mtaBuild script: this, buildTarget: 'NEO'
      }
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

