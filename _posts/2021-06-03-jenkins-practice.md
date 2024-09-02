---
layout: post
title: Jenkins Practice
date: 2021-06-23 09:41:00-0000
description: An summarize of usage of Jenkins as a package server.
tags: ci&cd
categories: ci&cd
---

# Jenkins

# Jenkins自动构建探索、实践

## [Jenkins](https://www.jenkins.io/)简介

Jenkins是

1. CI/CD工具
2. Automation server
3. MIT开源许可

## 目标

了解移动端Jenkins实践过程，具备Jenkins维护、开发能力

1. 了解Jenkins服务搭建、部署过程
2. 掌握创建Jenkins任务的方法(Classic UI & pipeline)
3. 了解Groovy脚本语言；掌握Jenkins pipeline DSL；掌握Declarative和Scripted类型pipeline及区别;
4. 掌握常用step：sh, archiveArtifacts, dir等
5. 掌握安装、使用Jenkins插件的方法
6. 掌握通过Jenkins管理及使用凭据的方法
7. 掌握Jenkins的授权管理策略
8. 掌握Jenkins多节点管理、使用方法

## 安装、部署

[Installing Jenkins](https://www.jenkins.io/doc/book/installing/)

```bash

brew install jenkins-lts

brew services start jenkins-lts
```

### 关键模块

- [Jetty](https://www.eclipse.org/jetty/) provides a web server and servlet container
- Jenkins-WAR模块维护一个Jenkins servlet，把所有请求转交给Stapler进行路由分发
- Stapler把请求映射到Jenkins-Core模块
- Jenkins-Core提供核心功能

### 工作目录

```bash
➜  ~ tree ~/.jenkins -L 1
/Users/[user-name]/.jenkins
├── caches
├── config.xml
├── hudson.model.UpdateCenter.xml
├── hudson.plugins.git.GitTool.xml
├── identity.key.enc
├── jenkins.install.InstallUtil.installingPlugins
├── jenkins.install.InstallUtil.lastExecVersion
├── jenkins.install.UpgradeWizard.state
├── jenkins.model.JenkinsLocationConfiguration.xml
├── jenkins.telemetry.Correlator.xml
├── jobs
├── logs
├── nodeMonitors.xml
├── nodes
├── org.jenkinsci.plugins.workflow.flow.FlowExecutionList.xml
├── plugins
├── queue.xml
├── queue.xml.bak
├── secret.key
├── secret.key.not-so-secret
├── secrets
├── updates
├── userContent
├── users
├── war
├── workflow-libs
└── workspace
```

```bash
➜  ~ tree ~/.jenkins/workspace
/Users/[username]/.jenkins/workspace
├── Demo
│   └── demo.txt
└── Demo@tmp

2 directories, 1 file
```

```bash
➜  ~ tree ~/.jenkins/jobs -L 3
/Users/[username]/.jenkins/jobs
└── Demo
    ├── builds
    │   ├── 1
    │   ├── 2
    │   ├── 3
    │   ├── legacyIds
    │   └── permalinks
    ├── config.xml
    └── nextBuildNumber

5 directories, 4 files
```

## 创建任务

### Classic UI

![Untitled](https://user-images.githubusercontent.com/25997299/120635444-0f45aa00-c49f-11eb-8af4-1ba825f5d4c6.png)

### Pipeline

Jenkins Configuration as Code

```groovy
pipeline {
![jenkinsfile](https://user-images.githubusercontent.com/25997299/120635727-63e92500-c49f-11eb-8175-80d3492e348b.png)

    agent any
    parameters {
        string name: 'name', defaultValue: 'World'
    }

    stages {
        stage('Hello') {
            steps {
                 echo "Hello ${params.name}"
            }
        }
    }

     post {
          always {
            echo "Done"
          }
     }
}
```

按语法特点分为两种类型

1. Declarative DSL
2. Scripted

按文件位置分为两种类型

1. Jenkinsfile

![jenkinsfile](https://user-images.githubusercontent.com/25997299/120635768-6cd9f680-c49f-11eb-9844-11f851900cc4.png)

    ```groovy
    ➜  app_club_factory git:(feature/lauch-optimize) ✗ tree . -L 1
    .
    ├── Jenkinsfile
    ├── README.md
    ├── android
    ├── babel
    ├── babel.config.json
    ├── buz.config.js
    ├── buzplatform.config.js
    ├── commit.sh
    ├── common.json
    ├── createmoduleid.js
    ...
    ```

2. Standalone git repo

![standalone](https://user-images.githubusercontent.com/25997299/120635840-7bc0a900-c49f-11eb-893a-89d0f4c4835a.png)

    ```groovy
    ➜  jenkins_pipelines git:(master) ls jenkins
    Android-Package                       ExportOptions_Development_Prime.plist iOS-Package-Prime                     show_file_list.sh
    Android-RN-Production                 Old-driver                            iOS-RN-Production                     sync_data.py
    Android-RN-Staging                    archive.sh                            iOS-RN-Staging                        test.sh
    Deprecated                            convert_string_to_qrimage.js          iOS-sync-profile                      traceDownloadFail.py
    ExportOptions_AdHoc.plist             daily_jobs.py                         image_print.js                        upload_to_app_store.sh
    ExportOptions_AdHoc_Prime.plist       get_android_version.sh                incrent_ios_build_number.sh           upload_to_fir.sh
    ExportOptions_AppStore.plist          iOS-Hotfix-Production                 rn_migration.js                       upload_zip_to_s3.py
    ExportOptions_AppStore_B2B.plist      iOS-Hotfix-Test                       rn_osp_publish.js                     use_local_cer.rb
    ExportOptions_AppStore_Prime.plist    iOS-Package                           rn_publish.js
    ExportOptions_Development.plist       iOS-Package-B2B                       setup_tools.sh
    ```

    ```groovy
    ➜  jenkins_pipelines git:(master) ls jenkins/Android-Package
    HibossJenkinsfile Jenkinsfile
    ```

    ```groovy
    stage('fetch @ master') {
                    agent { label 'master' }
                    steps {
                      sh label: 'Resetting workspace', script: '''
                                  git reset --hard
                                  git clean -fd
                              '''
                      dir("source_code/app_club_factory") {
                          sh label: 'Resetting workspace', script: '''
                                  git reset --hard
                                  git clean -fd
                              '''
                      }
                      dir("source_code/app_club_factory") {
                          git branch: env.App_Club_Factory_Local_Branch, credentialsId: params.Git_Credential_Id, url: 'git@gitlab.yuceyi.com:client/app_club_factory.git'
                          script {
                              if (!fileExists(RN_TARGET)) {
                                  sh "git clone git@gitlab.yuceyi.com:client/ios_2c.git ios"
                              }
                          }
                          dir(RN_TARGET) {
                              sh label: 'Resetting workspace', script: '''
                                  git reset --hard
                                  git clean -fd
                                  git fetch --prune
                              '''
                              sh label: 'Checkout branch', script: "git checkout -B ${env.iOS_Local_Branch} --track ${env.iOS_Remote_Branch}"
                              sh label: 'Checkout branch', script: "git pull origin ${env.iOS_Local_Branch}"
                              sh label: 'Show latest change', script: "git log --pretty=oneline -n 10"
                          }
                      }

                    }
                  } // master
    ```

## Pipeline开发

### Groovy脚本语言

[Learn](https://groovy-lang.org/learn.html)

- Groovy对于Java开发人员来说很简单，因为Java和Groovy的语法非常相似。
- Groovy是一种基于Java平台的面向对象语言
- 您可以使用现有的Java库。
- 同时支持静态和动态类型。

### Declarative pipeline

```groovy
pipeline {
    agent any
    parameters {
        string name: 'name', defaultValue: 'World'
    }

    stages {
        stage('Hello') {
            steps {
								script {

								}
                 echo "Hello ${params.name}"
            }
        }
    }

     post {
          always {
            echo "Done"
          }
     }
}
```

Declarative pipeline展现出不同层次的特性

- DSL特性

Declarative pipeline是一个特定领域语言。对一些写法有特殊的要求，比如在stage中必需要有且只有一个steps；所有的step必须被一个steps包裹等。

[Pipeline DSL Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)

- Groovy特性

Declarative pipeline也表现出Groovy特性，即普通的Groovy代码在这里是可以直接运行的。但是这里会有Jenkins沙盒限制

- Jenkins特性

Jenkins插件引入的API，在这里都可以直接使用。

### Scripted pipeline

```groovy
node('master') {
    try {
        properties([
            parameters([
                string(name: 'name', defaultValue: 'World')
            ])
        ])

        stage('fetch @ slave4') {
            echo "Hello ${params.name}"
sh label: '', script: 'echo Hello '
        }
    }
    catch(_) {
        echo "Error"
    }
    finally {
        echo "Done"
    }

}
```

Scripted pipeline相比Declarative pipeline没有了DSL特性。它完全是一个Groovy脚本，可以调用在Jenkins沙盒内许可的任意的Api，也可以调用Jenkins插件引入的新功能；除此之外没有其它的限制。

## 几个特殊step及重要作用

1. sh

```groovy
Runs a Bourne shell script, typically on a Unix node. Multiple lines are accepted.

An interpreter selector may be used, for example: #!/usr/bin/perl

Otherwise the system default shell will be run, using the -xe flags (you can specify set +e and/or set +x to disable those).
```

作为pipeline与其它生态系统之间的一个桥梁。

```groovy
// 更新Jenkins脚本代码；更新业务代码
def fetchStages() {
    def stages = [:]
    for (aNode in avaliableNodes()) {
    def stageName = "fetch @${aNode}"
    def nodeName = "${aNode}"
    stages[stageName] = {
        node(nodeName) {
        stage(stageName) {
            if (nodeName != 'master' && env.Should_Do_RN_Build == 'false') { //跳过RN打包
            print("Skipped due to SkipRNBuild")
            } else {
            checkout(scm) //更新Jenkins pipeline代码
            sh(label: 'Preparing repo', script: "yarn fetch ${env.WORKSPACE}/source_code ${env.App_Club_Factory_Local_Branch} '' ${env.iOS_Local_Branch}") //更新业务代码
            }
        }
        }
    }
    }
    stages['failFast'] = true
    return stages
}
```

2. archiveArtifacts

   ```groovy
   dir("source_code/app_club_factory/${RN_TARGET}/bundle") {
             archiveArtifacts "bundle.zip"
         }
   ```

3. dir　更换工作目录

   ```groovy
   stage('build @ slave1') {
             node('slave1') {
               dir("source_code/app_club_factory") {
                   sh(label: '... BUILDING ...',
                           script: """
                   ...
                 )
                 // sh label: 'Build js bundle', script: "npm run ${RN_SCRIPT}"
               }
             }
           } // build @ slave1
   ```

## Jenkins插件

通过Web UI搜索、安装插件

![plugin](https://user-images.githubusercontent.com/25997299/120635885-8aa75b80-c49f-11eb-9a34-cd06036497ad.png)

在Pipeline中使用插件

```groovy
parameters {
          booleanParam defaultValue: true, description: '是否要跳过RN打包阶段;该选项在archive_type为AppStore时失效', name: 'SkipRNBuild'

          listGitBranches branchFilter: '.*', credentialsId: 'ce9fe0a2-ce19-406a-b825-5c13cff6d8e6', defaultValue: '', name: 'app_club_factory_branch', quickFilterEnabled: true, remoteURL: 'git@gitlab.yuceyi.com:client/app_club_factory.git', selectedValue: 'NONE', sortMode: 'ASCENDING_SMART', tagFilter: '.*', type: 'PT_BRANCH', description: '选择 app_club_factory 仓库分支。默认为 master 分支'

          ...
      }
```

## 凭证管理

在Web UI中创建、管理凭证

![credentials](https://user-images.githubusercontent.com/25997299/120635904-9004a600-c49f-11eb-80a9-94aa79f12906.png)

在Pipeline中使用凭证

```groovy
git branch: env.App_Club_Factory_Local_Branch, credentialsId: params.Git_Credential_Id, url: 'git@gitlab.yuceyi.com:client/app_club_factory.git'
```

```groovy
withCredentials([usernamePassword(credentialsId: params.apple_id, passwordVariable: 'password', usernameVariable: 'username')]) {
                              sh(label: "Uploadting ipa to app store", script: "sh jenkins/upload_to_app_store.sh ${ipa_path} ${username} ${password}")
                          }
```

## 授权策略

使用[Role-based Authorization Strategy](https://plugins.jenkins.io/role-strategy/)插件进行基于角色的权限管理

1. 创建角色

   指定角色权限范围

2. 管理角色

   为用户分派角色

特殊的用户组

- Anonymous匿名用户
- authenticated授权用

## 多节点管理

通过Web UI创建节点

![node](https://user-images.githubusercontent.com/25997299/120635921-9561f080-c49f-11eb-88e1-cbc1c2600545.png)

配置SSH免密登陆

1. 在master节点生成或使用已有的公、私钥对
2. 把master节点公钥对追加到slave节点的known_hosts文件中

在Pipeline中为任务指定节点

```groovy
stage('build @ slave3') {

                ...
                agent { label 'slave3' }
                steps {
								...
                 }
              } // build @ slave3
```

## Shared Library

Shared Library是Jenkins提供的多任务代码复用方案。它要求项目结构如下：

```bash
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```

- src

  包含Groovy源码，在运行时Jenkins会添加对应的classpath来保持类的正常识别；

- vars

  用来定义全局变量或扩展新的step供pipeline使用

  定义全局变量：

  ```bash
  vars/log.groovy
  def info(message) {
      echo "INFO: ${message}"
  }

  def warning(message) {
      echo "WARNING: ${message}"
  }
  ```

  使用全局变量，以单例形式

  ```bash
  Jenkinsfile
  @Library('utils') _

  log.info 'Starting'
  log.warning 'Nothing to do!'
  ```

  自定义step

  ```bash
  // vars/sayHello.groovy
  def call(String name = 'human') {
      // Any valid steps can be called from this code, just like in other
      // Scripted Pipeline
      echo "Hello, ${name}."
  }
  ```

  使用自定义step

  ```bash
  sayHello 'Joe'
  sayHello() /* invoke with default arguments */
  ```

SharedLibray存在的问题，不能引用三方库

While possible, accessing third-party libraries using @Grab from trusted libraries has various issues and is not recommended. Instead of using @Grab, the recommended approach is to create a standalone executable in the programming language of your choice (using whatever third-party libraries you desire), install it on the Jenkins agents that your Pipelines use, and then invoke that executable in your Pipelines using the bat or sh step.

## 工具链

随着Pipeline变得越来越复杂，依赖Pipeline snippet generator来生成我们需要的指定显得越来越笨重，同时测试一小段代码需要运行整个Pipeline显得很低效，如果因为一个单词的书写错误我们不得不修改后再次去运行整个Pipeline，这种开发体验是很差劲的。

解决以上问题我们可以从两方面着手：

1. 寻找合适的IDE。

   我们可以从syntax highlighting、code completion、 go to definition、lint等几个方面来衡量。在Jenkins pipeline开发模式下主要语言是Pipeline DSL和Groovy

   - Pipeline DSL

     Jenkins提供了一个GDSL文件来描述该DSL供我们导入IDE来使用；

![dsl](https://user-images.githubusercontent.com/25997299/120635944-9dba2b80-c49f-11eb-8138-922bf825da5b.png)

![demo](https://user-images.githubusercontent.com/25997299/120635957-a1e64900-c49f-11eb-8e33-084edfbdce64.png)

    - Groovy

        因为流行的[VS Code并不支持code completion](http://groovy-lang.org/ides.html)，看起来[IntelliJ IDEA](https://www.jetbrains.com/idea/)是一个不错的选择

        通过实践发现，Jenkins对Groovy三方库支持力度不够，我们很多业务逻辑是需要使用其它类型代码如JS来完成的。所以不见得IntelliJ IDEA有最佳的选择，要结合项目实际来定。

2. 完善单元测试

   存在两个维度的单元测试

   - Pipeline

     Pipeline中有许多能力是依赖Jenkins环境的，要想真正的测试一段代码，只能在Jenkins中去执行。

     不过一些基于mock的方案或许值得一试[Jenkins Pipeline Unit testing framework](https://github.com/jenkinsci/JenkinsPipelineUnit)

   - 业务代码

     建议将业务代码使用其它语言来写，这样可以摆脱Jenkins的各种缺陷；同时也可能更方便的进行单元测试。

# 最佳实践

1. 使用Pipeline而不使用Classic UI创建Jenkins任务

   配置即代码；使用pipeline可以更好的进行

   - 版本管理
   - 分支管理

   同时建议对环境、工具的管理也要尽可能的代码化。比如node package.json，ruby Gemfile，python requirements，shell中通过hash判断命令是否可用等

2. 如果有多个Jenkins任务，并且逻辑比较复杂，可以把它们放在一个Git repo中做进一步工程化管理。
3. 使用Scripted pipeline而不是Declarative pipeline

   Scripted pipeline的优势

   - 代码组织可以更灵活
   - Api能力更强

4. Groovy用作胶水层

   因为Groovy代码有以下副作用

   - Groovy代码运行在Controller，影响master性能
   - Groovy代码有沙盒限制
   - Jenkins缺少添加`classpath`的支持

   同时Shared Library不支持三方库

   所以建议把Groovy当做链接Jenkins和业务逻辑的胶水层

5. 选择一个有优势的语言而不是多个完成主业务逻辑；并完成单元测试
   - 把业务逻辑分散在太多的生态环境下会增加pipeline复杂度
   - 对业务逻辑进行单元测试覆盖，减少冗长的Jenkins测试流程引进的时间成本
6. 对pipeline进行单元测试

   [Jenkins Pipeline Unit testing framework](https://github.com/jenkinsci/JenkinsPipelineUnit)
