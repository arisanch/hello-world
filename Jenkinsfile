#!/usr/bin/env groovy
def VERSION = 'UNKNOWN'

pipeline {
  agent none
  environment {
    DOCKER_REPO_NAME = "cmays/hello-world"
  }

  tools {
    maven "M3"
  }

  stages {
    stage('Build') {
      agent {label 'kube-slave'}
      steps {
        container('jenkins-dind') {
          script  {
            VERSION = sh(script: 'mvn org.apache.maven.plugins:maven-help-plugin:3.1.0:evaluate -Dexpression=project.version -q -DforceStdout --batch-mode',returnStdout: true)
          }
        }
      }
    }

    stage('Make Container') {
      agent {label 'kube-slave'}
      steps {
        container('jenkins-dind') {
          sh "docker build -t ${DOCKER_REPO_NAME}:${VERSION} ."
        }
      }
    }

    stage('Push Container') {
      agent {label 'kube-slave'}
      when {
        branch 'master'
      }
      steps {
          container('jenkins-dind') {
          withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "docker login -u ${USERNAME} -p ${PASSWORD}"
            sh "docker push ${DOCKER_REPO_NAME}:${VERSION}"
            echo "${VERSION}"
          }
        }
      }
    }
  }
}
