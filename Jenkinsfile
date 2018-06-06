pipeline {
  agent any
  stages {
    stage ('编译代码，打包镜像，这里用nginx作了一个简单的镜像，省去了编译部分') {
      environment {
        IMG_NAME = "${JOB_NAME}"
        IMG_VERSION = "${CHANGE_ID}"
      }
      steps {
        sh 'echo ${JOB_NAME}:${IMG_VERSION}'
      }
    }
  }
}
