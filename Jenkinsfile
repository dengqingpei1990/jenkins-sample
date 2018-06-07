pipeline {
  agent any
  environment {
        PROJECT_NAME = env.JOB_NAME.substring(0, env.JOB_NAME.indexOf("/"))
  }
  stages {
    stage ('编译，打包，推送线下仓库') {
      when { not { branch 'master' } }
      environment {
        IMG_TAG = "v_${BUILD_ID}"
        IMG_NAME = "registry.example.com:5000/${PROJECT_NAME}"
      }
      steps {
        echo 'build code from SCM.....'
        script {
          def customImage = docker.build("${IMG_NAME}:${IMG_TAG}")
          customImage.push()
        }
      }
    }
    stage ('部署到测试环境') {
      when { not { branch 'master' } }
      environment {
        IMG_VERSION = "v_${BUILD_ID}"
        IMG_NAME = "registry.example.com:5000/${PROJECT_NAME}"
        // RES = sh returnStdout: true, script : "echo aaa"
      }
      steps {
        /** 
        * 这里用到了kubernetesDeploy步骤，由Kubernetes Continuous Deploy插件提供，需手动安装
        * 'kubeconfig'即为master节点下/etc/kubenetes/admin.conf文件内容，需在jenkins credentials设置中添加
        * 该插件执行kubectl apply -R -f k8s/*.yaml,并用环境变量替换yaml文件中的${XXX}变量，实现模板功能 
        */
        kubernetesDeploy configs: 'k8s/*.yaml', kubeConfig: [path: ''], kubeconfigId: 'kubeconfig', secretName: '', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']

      }
    }
  }
}


