#!/usr/bin/groovy
def envStage = "${env.JOB_NAME}-staging"
def envProd = "${env.JOB_NAME}-production"

node ('kubernetes'){

  git 'https://github.com/jmlambert78/golang-example'

  stage 'canary release'
    if (!fileExists ('Dockerfile')) {
      writeFile file: 'Dockerfile', text: 'FROM golang:onbuild'
    }

    def newVersion = performCanaryRelease {}

    def rc = getKubernetesJson {
      port = 8080
      label = 'golang'
      icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/gopher.png'
      version = newVersion
      imageName = clusterImageName
    }

  stage 'Rolling upgrade Staging'
    kubernetesApply(file: rc, environment: envStage)

  approve{
    room = null
    version = canaryVersion
    console = fabric8Console
    environment = envStage
  }

  stage 'Rolling upgrade Production'
    kubernetesApply(file: rc, environment: envProd)

}
