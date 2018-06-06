pipeline {
  agent any
  stages {
    stage ('编译代码，打包镜像，这里用nginx作了一个简单的镜像，省去了编译部分') {
      environment {
        IMG_NAME = ""
        IMG_VERSION = "${BUILD_ID}"
      }
      steps {
        script {
          echo "${env.IMG_NAME} == 2" //${env.JOB_NAME}.substring(0, ${env.JOB_NAME}.indexOf("/"))
        }
        sh 'echo $IMG_NAME'
      }
    }
  }
}
