import hudson.AbortException

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

@Library('piper-library-os') _

node() {
  stage("Clone sources and setup environment") {
    deleteDir()
    echo "MH-SCM: ${scm}"
    checkout scm
    setupCommonPipelineEnvironment script: this

  }
  stage("Build Fiori App"){
      echo 'Build'
  }

  stage("Deploy Fiori App"){
      echo 'Deploy'
  }
}

