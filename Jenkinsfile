import hudson.AbortException

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

@Library('piper-library-os') _

node() {
  stage("Clone sources and setup environment x") {
    deleteDir()
    def scmVars = checkout scm directory: 'marcus'
    echo "MH-SCM: ${scmVars}"
    setupCommonPipelineEnvironment script: this

  }
  stage("Build Fiori App"){
      echo 'Build xx'
  }

  stage("Deploy Fiori App"){
      echo 'Deploy xx'
  }
}

