library 'LEAD'
pipeline {
  agent none
  stages {
    stage('Initialization'){ 
      agent {label 'master'}
      steps {notifyPipelineStart()}
    }
    /// [build]
    stage('Build') {
      agent {
        label "lead-toolchain-skaffold"
      }
      steps {
        notifyStageStart()
        container('skaffold') {
          sh "skaffold build --file-output=image.json"
          stash includes: 'image.json', name: 'build'
          sh "rm image.json"
        }
      }
      post {
        success {
          notifyStageEnd()
        }
        failure {
          notifyStageEnd([result: "fail"])
        }
      }
    }
    /// [build]

    /// [stage]
    stage("Deploy to Staging") {
      agent {
        label "lead-toolchain-skaffold"
      }
      when {
          branch 'master'
      }
      environment {
        ISTIO_DOMAIN   = "${env.stagingDomain}"
      }
      steps {
        notifyStageStart()
        container('skaffold') {
          unstash 'build'
          sh "skaffold deploy -a image.json -n ${env.stagingNamespace}"
        }
      }
      post {
        success {
          notifyStageEnd([status: "Successfully deployed to staging:\nspringtrader.${env.stagingDomain}/spring-nanotrader-web/"])
        }
        failure {
          notifyStageEnd([result: "fail"])
        }
      }
    }
    /// [stage]

    /// [gate]
    stage ('Manual Ready Check') {
      agent none
      when {
        branch 'master'
      }
      options {
        timeout(time: 30, unit: 'MINUTES')
      }
      input {
        message 'Deploy to Production?'
      }
      steps {
        echo "Deploying"
      }
    }
    /// [gate]

    /// [prod]
    stage("Deploy to Production") {
      agent {
        label "lead-toolchain-skaffold"
      }
      when {
          branch 'master'
      }
      environment {
        ISTIO_DOMAIN   = "${env.productionDomain}"
      }
      steps {
        notifyStageStart()
        container('skaffold') {
          unstash 'build'
          sh "skaffold deploy -a image.json -n ${env.productionNamespace}"
        }
      }
      post {
        success {
          notifyStageEnd([status: "Successfully deployed to production:\nspringtrader.${env.productionNamespace}/spring-nanotrader-web/"])
        }
        failure {
          notifyStageEnd([result: "fail"])
        }
      }
    }
    /// [prod]
  }
  post {
    success {
      echo "Pipeline Success"
      notifyPipelineEnd()
    }
    failure {
      echo "Pipeline Fail"
      notifyPipelineEnd([result: "fail"])
    }
  }
}
