pipeline {
  agent any
  environment {
        PROJECT_NAME = env.JOB_NAME.substring(0, env.JOB_NAME.indexOf("/"))
  }
  stages {
    stage ('编译，打包，推送线下仓库') {
      when { not { branch 'master' } }
      environment {
        IMG_TAG = sh returnStdout: true, sh '''
        #/bin/bash
        if [ ! -e version.txt ]; then echo 1.0.0>>version.txt; fi
        head -n1 version.txt | awk -F '.' '{print $1"."$2"."$3+1}'
        '''
        IMG_NAME = "registry.example.com:5000/${PROJECT_NAME}"
      }
      steps {
        // 默认从project根目录中的Dockerfile打包镜像，这里用了nginx镜像，和一个简单的html，省去编译的步骤
        echo 'build code from SCM.....'
        script {
          echo '${IMG_TAG}'
          def customImage = docker.build("${IMG_NAME}:${IMG_TAG}")
          customImage.push()
        }
      }
    }
    stage ('部署到测试环境') {
      when { not { branch 'master' } }
      environment {
        IMG_TAG = "v_${BUILD_ID}"
        IMG_NAME = "registry.example.com:5000/${PROJECT_NAME}"
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
    stage ('部署到线上环境') {
      when { branch 'master' }
      environment {
        IMG_NAME = "registry.cn-shanghai.aliyuncs.com/dengqingpei/${PROJECT_NAME}"
      }
      steps {
        script {
          def tags = sh returnStdout: true, script : '''curl -s http://registry.example.com:5000/v2/jenkins-sample/tags/list | jq . | grep -E 'v_*' | tail -n 20 | tr -d ' ",' | sort -t '_' -k 2 -nr '''
          def pm = input message: '部署指定镜像版本到线上环境', id:'deployment', ok: '部署', submitterParameter: 'approval', parameters: [choice(choices: tags, description: '镜像版本，默认最新', name: 'tag')]
          env.IMG_TAG = pm.tag
          env.SUBMITTER = pm.approval
        }
        sh "echo ${SUBMITTER} 将 ${IMG_NAME}:${IMG_TAG} 发布到生产环境 !"
      }
    }
  }
}
