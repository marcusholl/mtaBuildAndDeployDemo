import hudson.AbortException

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

@Library('piper-library-os') _

node() {
  stage("Clone sources and setup environment") {
    deleteDir()
    checkout scm
    setupCommonPipelineEnvironment script: this
    prepareDefaultValues script: this

  }
  stage("Build Fiori App"){
        mtaBuild script: this, buildTarget: 'NEO'
  }

  stage("Deploy Fiori App"){
        neoDeploy script: this
  }
}

