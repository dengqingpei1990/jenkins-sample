pipeline {
  agent any
  environment {
    PROJECT_NAME = env.JOB_NAME.substring(0, env.JOB_NAME.indexOf("/"))
    LOCAL_REGISTRY = "registry.example.com:5000"   //线下镜像仓库,未使用tls身份认证
    REGISTRY = "registry.example.com:5000"         //线上镜像仓库
  }
  stages {
    stage ('编译') {
      when { not { branch 'master' } }
      steps {
        echo "compile code from SCM...."
      }
    }
    stage ('打包镜像,推送线下仓库') {
      when { not { branch 'master' } }
      environment {
        IMG_NAME = "${LOCAL_REGISTRY}/${PROJECT_NAME}"
      }
      steps {
        // 默认从project根目录中的Dockerfile打包镜像，这里用了nginx镜像，和一个简单的html，省去编译的步骤
        script {
          env.IMG_TAG = sh (returnStdout: true, script: '''
          #/bin/bash
          tag=$(head -n 1  version.txt)
          tag_list=$(curl -s -XGET http://${LOCAL_REGISTRY}/v2/${PROJECT_NAME}/tags/list | jq .tags)
          last_tag=$(echo $tag_list | tr -d '[]\n"' | awk -F ',' '{print $NF}')
          if [ "$(echo $tag_list | grep ${tag} | wc -l)" == "1" ]; 
            then echo "版本冲突！'${IMG_TAG}'已构建过,请修改version.txt文件更换版本号,上次构建成功的版本号:${last_tag}"&&exit 1
          fi
          echo $tag
          '''
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
