pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
  stages {
    stage('ビルド設定の作成') {
      when {
        expression {
          openshift.withCluster() {
              openshift.withProject("development-orval") {
                return !openshift.selector("bc", "orval").exists()
              }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("development-orval") {
              openshift.newBuild("--name=orval", "--image-stream=openshift/jdk8", "--binary")
            }
          }
        }
      }
    }
    stage('アプリケーションのテスト') {
      steps {
        script {
            openshift.withCluster() {
                sh "mvn test"
            }
            // Rocket.Chatへ通知
            rocketSend channel: 'orval-dev', message: 'complete test'
        }
      }
      post {
        failure {
            // Rocket.Chatへ通知
            rocketSend channel: 'olval-dev', message: currentBuild.currentResult
        }
      }
    }
    stage('コンテナのビルド') {
      steps {
        script {
            openshift.withCluster() {
                sh "mvn package"
                openshift.withProject("development-orval") {
                  openshift.selector("bc", "orval").startBuild("--from-file=target/orval.jar", "--wait")
                }
            }
        }
      }
    }
    stage('開発環境へのデプロイ設定') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("development-orval") {
              return !openshift.selector('dc', 'orval').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("development-shotodoke") {
              openshift.newApp("shotodoke:latest", "--name=shotodoke").narrow('svc').expose()
            }
          }
        }
      }
    }
  }
}