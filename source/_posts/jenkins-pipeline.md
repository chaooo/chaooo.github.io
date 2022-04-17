---
title: 「Jenkins」声明式流水线Jenkins Pipeline
date: 2020-04-06 11:56:53
tags: [Jenkins, 环境配置, Linux]
categories: 环境配置
---


### 1. Jenkins Pipeline 基本概念
流水线是用户定义的一个CD流水线模型 。流水线的代码定义了整个的构建过程, 他通常包括构建, 测试和交付应用程序的阶段 。
`Jenkins Pipeline`（或简称为"Pipeline"）是一套插件，将持续交付的实现和实施集成到Jenkins中。
持续交付`Pipeline`自动化的表达了这样一种流程：将基于版本控制管理的软件持续的交付。
Jenkins Pipeline 的定义通常被写入到一个`Jenkinsfile`文本文件中，该文件可以被放入项目的源代码控制库中。
<!-- more -->

### 2.Jenkinsfile 基础语法
``` json
pipeline {                // 特定语法，pipeline 块定义了整个流水线中完成的所有的工作
    agent any             // agent为整个流水线分配一个执行器 (在节点上)和工作区
    stages {              // 所有流程（状态）的外层块，仅有一个
        stage('Build') {  // 每个stage为一流程，定义名称
            steps {       // 步骤块，内部包含具体操作
                sh 'make' // sh操作，其引号间的文字会当成shell直接执行
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml'  //junit使用匹配的定义测试xml进行单元测试
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```


### 3.创建 Hello World 流水线
1. 登录`Jenkins`，新建任务(New Item)，选择`流水线`，输入工程名称`Hello Pipeline`，确定。
2. 填写描述，勾选`参数化构建过程(This project is parameterized)`。
3. 添加参数，选择`字符参数(String parameter)`，并设置这个字符串参数(名称,默认值,描述)，这样我们在Jenkinsfile中就可以取到这个值了。
4. 向下滑动到`流水线`，定义选择`Pipeline script`，脚本输入如下内容，然后保存。
``` json
pipeline {
    agent any
    environment {                //环境变量
        GREETING="Hello"
    }
    stages{
        stage('打招呼') {
            steps{
                sh 'echo "$GREETING $TITLE"'
            }
        }
   }
   post {                        //构建完成后置操作
        aborted {                //如果构建中断，则执行
            echo '构建被中止!'
        }
        success {                //构建成功执行
            echo '构建成功!'
        }
       failure {                 //构建失败执行
           echo '构建失败!'
       }
    }
}
```

5. 点击`Build with Parameters(参数化构建)`，然后`开始构建`。
6. 构建完成输出界面：
![构建完成输出界面](up-e49f9a3240d011242cd6093b055cf6709c7.webp "构建完成输出界面")


7. 把鼠标放在打招呼下边的绿色框上，点出现的`logs`，可以看到输出了预期的值。
8. 找到左下角的`Build History(构建历史)`的构建版本号，如当前是`#1`，点进去，选择`Console Output`查看详细的执行日志。
``` shell
# 成功Console Output
Started by user charles
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /home/jenkins/root/workspace/Hello Pipeline
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (打招呼)
[Pipeline] sh
+ echo 'Hello Jenkins Pipeline'
Hello Jenkins Pipeline
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
构建成功!
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```
