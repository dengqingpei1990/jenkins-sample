pipeline {
  agent any
  environment {
        IMG_NAME = env.JOB_NAME.substring(0, env.JOB_NAME.indexOf("/"))
  }
  stages {
    stage ('编译代码，打包镜像，推送线下仓库') {
      when { not { branch 'master' } }
      environment {
        IMG_VERSION = "v_${BUILD_ID}"
      }
      steps {
        // 默认从project根目录中的Dockerfile打包镜像，这里用了nginx镜像，和一个简单的html，省去编译的步骤
        echo 'build code from SCM.....'
        // credentials需要在jenkins上手动创建，即为docker registry的登陆用户和密码
        withDockerRegistry(credentialsId: 'docker-registry-local', url: 'https://registry.example.com:5000') {
          script {
            def customImage = docker.build("registry.example.com:5000/${IMG_NAME}:${IMG_VERSION}")
            customImage.push()
          }
        }
      }
    }
    stage ('部署到测试环境') {
      when { not { branch 'master' } }
      environment {
        IMG_VERSION = "v_${BUILD_ID}"
      }
      steps {
        /** 
        * 这里用到了kubernetesDeploy步骤，由Kubernetes Continuous Deploy插件提供，需手动安装
        * 'kubeconfig'即为master节点下/etc/kubenetes/admin.conf文件内容，需在jenkins credentials设置中添加
        * 该插件执行kubectl apply -R -f k8s/*.yaml,并用环境变量替换yaml文件中的${XXX}变量，实现模板功能 
        */
        kubernetesDeploy configs: 'k8s/*.yaml', dockerCredentials: [[credentialsId: 'docker-registry-local', url: 'https://registry.example.com:5000']], kubeConfig: [path: ''], kubeconfigId: 'kubeconfig', secretName: 'test', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']
      }
    }
  }
}


