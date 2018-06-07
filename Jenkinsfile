pipeline {
  agent any
  environment {
        IMG_NAME = env.JOB_NAME.substring(0, env.JOB_NAME.indexOf("/"))
  }
  stages {
    stage ('编译代码，打包镜像，这里用nginx作了一个简单的镜像，省去了编译部分') {
      environment {
        IMG_VERSION = "${BUILD_ID}"
      }
      steps {
        withDockerRegistry(credentialsId: 'docker-registry', url: 'https://registry.cn-shanghai.aliyuncs.com') {
          script {
            def customImage = docker.build("registry.cn-shanghai.aliyuncs.com/dengqingpei/${IMG_NAME}:${IMG_VERSION}")
          }
        }
      }
    }
  }
}
