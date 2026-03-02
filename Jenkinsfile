String registryEndpoint = 'evergiven.ury.york.ac.uk:5000'

def branch = env.BRANCH_NAME.replaceAll("/", "_")
def image
String imageName = "stv-web:${branch}-${env.BUILD_ID}"

pipeline {
  agent {
    label 'docker'
  }

  environment {
    REGISTRY_ENDPOINT = credentials('docker-registry-endpoint')
  }

  stages {
    stage('Build image') {
      steps {
        script {
          GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
          docker.withRegistry('http://' + registryEndpoint, 'docker-registry') {
            image = docker.build(imageName, "--build-arg STV_WEB_VERSION_ARG=${env.BRANCH_NAME}-${env.BUILD_ID} --build-arg STV_WEB_COMMIT_ARG=${GIT_COMMIT_HASH} .")
          }
        }
      }
    }

    stage('Push image to registry') {
      steps {
        script {
          docker.withRegistry('https://' + registryEndpoint, 'docker-registry') {
            image.push()
            if (env.BRANCH_IS_PRIMARY) {
              image.push('latest')
            }
          }
        }
      }
    }
  }
}
