pipeline {
    agent {
      label "jenkins-gradle"
    }
    environment {
      ORG               = 'jenkinsx'
      APP_NAME          = 'spring-boot-web-gradle'
      GIT_CREDS         = credentials('jenkins-x-git')
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GIT_USERNAME      = "$GIT_CREDS_USR"
      GIT_API_TOKEN     = "$GIT_CREDS_PSW"
    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
          PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
          HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        }
        steps {
          container('gradle') {
            // TODO 
            //sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
            sh "gradlew package"
            sh "docker build -t \$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:\$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:$PREVIEW_VERSION ."
            sh "docker push \$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:\$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:$PREVIEW_VERSION"
          }

          dir ('./charts/preview') {
           container('gradle') {
             sh "make preview"
             sh "jx preview --app $APP_NAME --dir ../.."
           }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
        steps {
          container('gradle') {
            // ensure we're not on a detached head
            sh "git checkout main"
            // until we switch to the new kubernetes / jenkins credential implementation use git credentials store
            sh "git config --global credential.helper store"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
            // TODO
            //sh "mvn versions:set -DnewVersion=\$(cat VERSION)"
          }
          dir ('./charts/spring-boot-web-gradle') {
            container('gradle') {
              sh "make tag"
            }
          }
          container('gradle') {
            sh 'gradle clean assemble'

            sh "docker build -t \$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:\$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
            sh "docker push \$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:\$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)"
          }
        }
      }
      stage('Promote to Environments') {
        when {
          branch 'master'
        }
        steps {
          dir ('./charts/spring-boot-web-gradle') {
            container('gradle') {
              sh 'jx step changelog --version \$(cat ../../VERSION)'

              // release the helm chart
              sh 'make release'

              // promote through all 'Auto' promotion Environments
              sh 'jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
  }
