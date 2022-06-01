// 所有命令放入到 Pipeline 中
pipeline {
  // 指定使用哪一个 Jenkins 节点
  agent any

  // 声明全局变量
  environment {
    harborName='stillmoon'
    harborPassword='YTT&wrq0208'
    harborAddress='back.stillmoon.top:8910'
    harborRepo='repo'
  }

  // 阶段
  stages {
    stage('拉取 git 仓库代码'){
      steps{
        checkout([$class: 'GitSCM', branches: [[name: '${tag}']], extensions: [], userRemoteConfigs: [[credentialsId: 'd1a5d26f-55e4-4fa3-b1ea-4562fd883230', url: 'git@github.com:PeatBoy/CI-CD_Test.git']]])
      }
    }
    stage('使用 Maven 构建项目'){
      steps{
        sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
      }
    }
    stage('使用 SonarQube 对代码质量检测'){
      steps{
        sh '/var/jenkins_home/sonar-scanner/bin/sonar-scanner -Dsonar.source=./ -Dsonar.projectname=${JOB_HOME} -Dsonar.projectKey=${JOB_NAME} -Dsonar.java.binaries=./target/ -Dsonar.login=ab403359436fe147275da1f499e54c6c45137f8e'
      }
    }
    stage('制作 Docker 镜像'){
      steps{
        sh '''mv ./target/*.jar ./docker/
docker build -t ${JOB_NAME}:${tag} ./docker'''
      }
    }
    stage('推送镜像到 Harbor 仓库'){
      steps{
        docker login -u ${harborName} -p ${harborPassword} ${harborAddress}
docker tag ${JOB_NAME}:${tag} ${harborAddress}/${harborRepo}/${JOB_NAME}:${tag}
docker push ${harborAddress}/${harborRepo}/${JOB_NAME}:${tag}
      }
    }
    stage('通知应用服务器部署镜像'){
      steps{
        sshPublisher(publishers: [sshPublisherDesc(configName: 'self', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "deploy.sh $harborAddress $harborRepo $JOB_NAME $tag $container_port $host_port", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      }
    }

  }

  post {
    success{
      dingtalk(
        robot: 'Jenkins-DingDing'
        type:'MARKDOWN'
        title:"success: ${JOB_NAME}"
        text: ["- 成功构建：${JOB_NAME}! \n- 版本：${tag} \n- 持续时间：${currentBuild.durationString}"]
      )
    }
    failure{
      dingtalk(
        robot: 'Jenkins-DingDing'
        type:'MARKDOWN'
        title:"success: ${JOB_NAME}"
        text: ["- 构建失败：${JOB_NAME}! \n- 版本：${tag} \n- 持续时间：${currentBuild.durationString}"]
      )
    }
  }
}