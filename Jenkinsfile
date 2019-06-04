import hudson.AbortException

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

@Library('piper-library-os') _

node() {
  stage("Clone sources and setup environment x") {
    deleteDir()
    checkout scm
    setupCommonPipelineEnvironment script: this
    
    dir('jenkins-pipelines') {
      git url: 'https://github.com/SAP/jenkins-pipelines'
    }
    
    dir('xcode-maven-plugin') {
      git url: 'https://github.com/marcusholl/xcode-maven-plugin.git', branch: 'l-test'
    }

  }
  stage("Build Fiori App"){
      echo 'Build xx'
  }

  stage("Deploy Fiori App"){
      echo 'Deploy xx'
  }
}

