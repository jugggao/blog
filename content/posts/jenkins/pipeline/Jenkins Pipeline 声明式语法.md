+++
title = "Jenkins Pipeline - 声明式语法"
description = "Jenkins Pipeline"
date = 2020-11-16
categories = ["Jenkins"]
tags = ["Jenkins", "Jenkins Pipeline"]
draft = false
+++

### Jenkins Pipeline 声明式语法

Jenkins Pipeline 有两种语法风格：

- 声明式管道语法（Declarative pipeline syntax）
- 脚本式管道语法（Scripted pipeline syntax）



脚本式管道语法以 Groovy 语言为基础，语法结构同 Groovy 相同。
由于 Groovy 不适合所有初学者。所以 Jenkins 团队为编写 Jenkins 流水线提供一种更简单的声明式管道。


这两种语法的区别在于语法和灵活性：

- 声明式管道语法鼓励采用[声明式编程模型](https://en.wikipedia.org/wiki/Declarative_programming)；脚本式管道语法遵循[命令式编程模型](https://en.wikipedia.org/wiki/Imperative_programming)。
- 声明式管道通过更严格和预定义的结构对用户施加了限制，这对于比较简单的持续交付流程来说是更理想的选择；而脚本式管道对于语法和结构唯一的限制是由 Groovy 子集本身定义的，脚本式管道几乎可以做任何事情，因此对于有更复杂的要求的用户或者对 Groovy 语言很熟练的用户来说是理想的选择。



**关于如何选择声明式语法或者脚本式语法？**

- 如果是 Jenkins 新手可以通过声明式管道语法入门，声明式管道提供了一些保护栏（框架），因此在刚开始学习的时候不会总出错，而且调试起来比较容易。另外，声明式管道的官方文档更详细。
- 如果是 Jenkins 高级用户或者对 Groovy 语言使用熟练，在工作中面对复杂的持续交付的场景，并且需要将逻辑复杂的管道代码模块化，那么脚本式管道结合共享库会更适合你。
- 如果是 Jenkins 管理员，则可能更希望坚持使用声明式管道，可以为使用者提供最佳的整体体验。并且能使其他的新同事更容易加入 Jenkins Pipeline 编写工作当中。

总之，仁者见仁，智者见智。无论怎么选择，只需要确保**一致性**即可。


那就先从简单、易读的声明式语法说起。

<!--more-->

## 1. 框架


在 vscode 中安装了 Jenkinsfile Support 插件后，输入 pipeline 回车会自动补全以下内容：


```groovy
pipeline{
    agent{
        label "node"
    }
    stages{
        stage("A"){
            steps{
                echo "========executing A========"
            }
            post{
                always{
                    echo "========always========"
                }
                success{
                    echo "========A executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
```
这是一个声明式语法的基础框架，其中包含了 4 个部分：

- **管道块（pipeline block）**
   - **pipline**：用于声明表示流水线的标识。
- **节（sections）**
   - **agent：**用于指定当前流水线将要执行的位置。
   - **post**：根据流水线或实际构建阶段的完成情况而运行指定的步骤。
   - **stages**：用于表示流水线各个步骤的声明。
   - **steps**：用于指定实际构建阶段的具体步骤。
- **指令（directives）**
   - **environment**：用于设置环境变量。
   - **options**：用于配置 Jenkins 应用本身的一些配置项。
   - **parameters**：用于配置参数化构建的参数列表。
   - **triggers**：用于定义流水线触发的一些机制与条件。
   - **stage**：用于表示实际构建的阶段，每个阶段执行不同的任务。
   - **tools**：用于定义部署流程中常用的一些工具。
   - **input**：用于在构建过程中指定交互式处理的步骤。
   - **when**：用于指定允许流水线根据特定的条件决定是否执行的阶段。
- **步骤（steps）**
   - Jenkins 提供了声明式语法可以引用的[所有步骤的列表](https://jenkins.io/doc/pipeline/steps/)。
   - **script**：用于执行脚本式管道语法步骤。





接下来对这 4 个部分逐步简单说明并举例。


> 接下来只对我日常会用到的一些指令或参数做解释并举例。
> 不会对[官方文档](https://jenkins.io/doc/book/pipeline/syntax/)做完整的翻译。



## 2. 管道块（pipeline block）


### 2.1 Pipeline


所有有效的声明式管道都必须包含在 pipeline 块中。


```groovy
pipeline {
  /* 插入声明式语法管道 */
}
```


pipeline 块表示这里将采用声明式的语法风格，不包含其他的特殊含义。
pipeline 块只能由节段（sections）、指令（directives）、指令（directives）或赋值语句组成。
属性引用语句被视为无参方法调用，例如 `input` 视为 `input()`。


## 3. 节（sections）


### 3.1 [agent](https://jenkins.io/doc/book/pipeline/syntax/#agent)


`agent` 部分表示当前流水线在什么环境下运行。


允许出现的位置： `pipeline` 块和 `stage` 阶段中。


常见的使用情况如下：


#### 场景一：使用 node 指定从节点执行任务

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587207650803-8eb81701-2def-4e6c-a8e9-0420c050fa0a.png#align=left&display=inline&height=73&margin=%5Bobject%20Object%5D&name=image.png&originHeight=73&originWidth=927&size=13563&status=done&style=stroke&width=927)

```groovy
// 指定在 ansible 节点上运行 ansible playbook
pipeline {
    agent {
        node {
            label 'ansible'
            customWorkspace '/data/jenkins/mysql-transfer'
        }
    }
    stages {
        stage('备份数据') {
            steps {
                ansiblePlaybook(
                    playbook: 'mysql-transfer.yaml',
                    inventory: 'hosts',
                    credentialsId: 'sample-ssh-key',
                    tags: 'mysql_backup'
                )
            }
        }
    }
}
```


#### 场景二：使用 docker 在容器中执行任务
```groovy
// 使用 docker 编译前端代码
pipeline {
    agent {
        docker {
            lable 'docker-jnlp-node'
            image 'harbor.ambow.com/jenkins/jnlp-node-slave'
        }
    }
    stages {
        stage('构建') { 
            steps {
                sh 'npm install --registry=https://registry.npm.taobao.org' 
            }
        }
    }
}
```


#### 场景三：不同的步骤使用不同的容器
```groovy
// 构建阶段使用 jnlp-node-slave 容器
// 部署阶段使用 ansible 容器
pipeline {
    agent none
    stages {
        stage('构建') { 
            agent {
                docker {
                    image 'harbor.ambow.com/jenkins/jnlp-node-slave'
                    args '-v /root/.npm:/root/.npm'
                }
            }
            steps {
                sh 'npm install --registry=https://registry.npm.taobao.org' 
            }
        }
        stage('部署'){
            agent {
                docker {
                    image 'harbor.ambow.com/tools/ansible'
                }
            }
            steps {
                sh 'echo "通过 ansbile 进行部署"' 
            }
        }
    }
}
```


> 使用容器时，Jenkins 会默认将 `$WORKSPACE` 挂载至容器中，因此各种操作直接基于当前目录即可。



#### 场景四：使用 kubernetes 启动临时 Pod 运行任务
```groovy
pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
kind: Pod
apiVersion: v1
metadata:
  name: jnlp
  namespace: jenkins
  labels:
    app: jnlp-slave
spec:
  containers:
  - name: jnlp
    image: harbor.ambow.com/jenkins/jnlp-slave:latest
    workingDir: /home/jenkins
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
      readOnly: true
    - mountPath: /home/jenkins/.kube
      name: kubectl-admin
      readOnly: true
    - mountPath: /home/jenkins/agent/workspace
      name: workspace-maven-data
      subPath: workspace
    - mountPath: /root/.m2
      name: workspace-maven-data
      subPath: .m2
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
  - name: kubectl-admin
    persistentVolumeClaim:
      claimName: kubectl-admin-pvc
  - name: workspace-maven-data
    persistentVolumeClaim:
      claimName: jenkins-pvc
"""
        }
    }
}
```


### 3.2 [post](https://jenkins.io/doc/book/pipeline/syntax/#post)


`post` 部分定义一个或多个 `steps`，而 `setps` 中定义的步骤则取决于流水线或阶段完成状态。


允许出现的位置： `pipeline` 块和 `stage` 块内。


常见的状态有：

- **always**：无论流水线或阶段的完成状态如何，都允许在 `post` 部分运行该步骤。
- **changed**：只有当前流水线或阶段的完成状态与它之前的运行不同时，才允许在 `post` 部分运行该步骤。
- **failure**：只有当前流水线或阶段完成状态为「failure」，才运行该步骤。通常 WebUI 为红色。
- **success**：只有当前流水线或阶段完成状态为「success」，才运行该步骤。通常 WebUI 为蓝色或绿色。
- **unstable**：只有当前流水线或阶段完成状态为「unstable」，才运行该步骤。通常由于测试失败，代码违规等情况造成。通常 WebUI 为黄色。
- **unsuccessful**：只要当前流水线或阶段完成状态不为「success」，才运行该步骤。WebUI 不是蓝色或绿色。
- **aborted**：只有当前流水线或阶段完成状态为「aborted」，才运行在 post 部分运行该步骤。通常由于流水线被手动关闭或者在交互式过程中被关闭。通常 WebUI 为灰色。

还有几种不太常用的状态，如 **fixed**、**regression**、**cleanup**。


#### 场景一：流水线完成时执行清理工作，仅当执行失败时发送邮件通知
```groovy
pipeline {
    agent {
        docker {
            lable 'docker-jnlp-node'
            image 'harbor.example.com/jenkins/jnlp-node-slave'
        }
    }
    stages {
        stage('构建') { 
            steps {
                sh 'npm install --registry=https://registry.npm.taobao.org' 
            }
        }
    }
    post {
        always {
            echo "Post：清理工作"
            sh "echo '执行一些清理工作'"
        }
        failure {
            echo "Post：邮件通知"
            mail to: "${MAIL}", from: "Jenkins<jenkins@test.com>",
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}
```


### 3.3 [stages](https://jenkins.io/doc/book/pipeline/syntax/#stages)


`stages` 部分包含一个或多个 `stage` 指令。流水线各阶段的声明，表示实际构建的各个阶段需要在 `stages` 块中定义。


允许出现的位置： `pipeline` 块内并且只能出现一次。


### 3.4 [steps](https://jenkins.io/doc/book/pipeline/syntax/#steps)


`steps` 部分包含一个或多个步骤。流水线实际运行的步骤的声明，表示某一个阶段中实际运行的各个步骤需要在 `steps` 块中定义。


允许出现的位置： `stage` 块内并且每个 `stage` 块内只能出现一次。


## 4. 指令


### 4.1 [environment](https://jenkins.io/doc/book/pipeline/syntax/#environment)


`environment` 指令用于指定环境变量，在不同区域指定，作用的范围也不同。


该指令支持一种特殊方法 `credentials()`，该方法可用于在 Jenkins 环境中通过标识符访问预定义的凭据，支持的凭据有以下几种：

- **Secret Text**：引用 Jenkins 凭据中 `Secret text` 类型的凭据作为环境变量。
- **Secret File**：引用 Jenkins 凭据中 `Secret file` 类型的凭据作为环境变量。
- **Username and password**：引用 Jenkins 凭据中 `Username with password` 类型的凭据作为环境变量，并且会自动定义两个额外的环境变量： `MYVARNAME_USR` 和 `MYVARNAME_PSW`。
- **SSH with Private Key**：引用 Jenkins 凭据中 `SSH Username with private key` 类型的凭据作为环境变量，并且可能会自动定义两个额外的环境变量：`MYVARNAME_USR` 和 `MYVARNAME_PSW`。



允许出现的位置：`pipeline` 块内或 `stage` 指令内。


#### 场景一：项目类似的情况下设置环境变量用于代码复用，更加清晰与简化流水线内容
```groovy
pipeline {
    agent {
        node {
            lable 'ansible'
        }
    }
    environment {
        serviceName = "gateway"
        servicePort = "30000"
        imageRepo = "harbor.example.com/base-service"
    }
    stages {
        stage('构建镜像') {
            steps {
                script {
                    docker.withRegistry('https://harbor.example.com', 'Harbor-Admin') {
                        def Image = docker.build("${imageRepo}/${serviceName}:${env.BUILD_ID}")
                        Image.push()
                    }
                }
            }
        }
    }
}
```
这样，对于类似的构建流程，我们只需要替换变量就可以进行代码复用了。


> 这个例子中使用了 `script` 步骤， `script` 步骤中可执行脚本式管道语法。



#### 场景二：使用凭据中的用户名密码作为环境变量


首先创建一个 Username with password 类型的全局凭据：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587380499564-9eb62191-f5d4-4a41-b7cf-bd9d2f175d17.png#align=left&display=inline&height=86&margin=%5Bobject%20Object%5D&name=image.png&originHeight=94&originWidth=1531&size=19463&status=done&style=stroke&width=1408)

然后在将凭据配置为环境变量：
```bash
pipeline {
    agent any
    stages {
        stage('Example Username/Password') {
            environment {
                HARBOR_ADMIN = credentials('Harbor-Admin')
            }
            steps {
                sh 'echo "Harbor-Admin user is $HARBOR_ADMIN_USR"'
                sh 'echo "Harbor-Admin is $HARBOR_ADMIN_PSW"'
            }
        }
    }
}
```
#### 场景三：使用凭据中的文件作为环境变量
首先创建一个 Secret file  类型的全局凭据：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587439274935-07b38752-f152-456d-a8c2-c55e58b1bf7a.png#align=left&display=inline&height=37&margin=%5Bobject%20Object%5D&name=image.png&originHeight=40&originWidth=1531&size=8761&status=done&style=stroke&width=1408)

```groovy
pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
kind: Pod
apiVersion: v1
metadata:
  name: jnlp
  namespace: jenkins
  labels:
    app: jnlp-slave
spec:
  containers:
  - name: jnlp
    image: harbor.ambow.com/jenkins/jnlp-slave:latest
    workingDir: /home/jenkins
    tty: true
"""
    	}
    }
    stages {
        stage('Example Secret file') {
            environment {
                KUBECONFIG_DEV = credentials('Kubeconfig-Dev')
            }
            steps {
                sh 'kubectl get pods --kubeconfig=${KUBECONFIG_DEV}'
            }
        }
    }
}
```


### **4.2 [options](https://jenkins.io/doc/book/pipeline/syntax/#options)**


`options` 指令用于配置 Jenkins Job 本身的配置，第一次运行后才会加载配置。


允许出现的位置： `pipeline` 块内只能出现一次， `stage` 指令内只能配置某些选项。


常用配置如下：

- **buildDiscarder**：保存最近历史构建记录的数量，可以节省 Jenkins 存储空间。
例如：`options { buildDiscarder(logRotator(numToKeepStr: '1')) }`
- **checkoutToSubdirectory**：在工作空间的子目录中检出源码。
例如： `options { checkoutToSubdirectory('dir') }`
- **disableConcurrentBuilds**：不允许并发执行流水线，防止同时访问共享资源。
例如： `options { disableConcurrentBuilds() }`
- **disableResume**：如果 Jenkins Master 重启，不允许恢复中断的流水线任务。
例如： `options { disableResume() }`
- **skipDefaultCheckout**：在 `agent` 节中，跳过从源代码控制中检出的默认情况。
例如： `options { skipDefaultCheckout() }`
- **skipStagesAfterUnstable**：在构建状态变为 `Unstable` 的情况下，跳过该阶段。
例如： `options { skipStagesAfterUnstable() }`
- **timeout**：设置流水线超时时间，如果超时则 Jenkins 将中止此流水线，状态为失败。可用的单位有 `HOURS`、`MINUTES`、`SECONDS`。
例如： `options { timeout(time: 1, unit: 'HOURS') }`
- **timestamps**：打印日志带上对应时间戳。
例如：`options { timestamps() }`
- **retry**：设置此流水线失败后的重试次数。
例如： `options { retry(3) }`



在 `stage` 指令内可以配置的选项有：

- **skipDefaultCheckout**
- **timeout**
- **retry**
- **timestamps**



#### 场景一：常用选项
```groovy
options {
    timestamps()
    disableConcurrentBuilds()
    timeout(time: 10, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))  
}
```


#### 场景二：不使用默认代码检出，自定义代码检出
```groovy
pipeline {
    agent any
    options {
        disableConcurrentBuilds()
        timestamps()
    }
    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }
    }
}        
```


#### 场景三：设置某一阶段的超时时间与重试次数
```groovy
pipeline {
    agent any
    stages {
        stage('编译代码') {
            options {
                timeout(time: 10, unit: 'MINUTES') 
                retry(3)
            }
            steps {
                sh "mvn clean install"
            }
        }
    }
}    
```


### 4.3 [parameters](https://jenkins.io/doc/book/pipeline/syntax/#parameters)


`parameters` 指令用于指定用户在触发 Pipeline 时应提供的参数列表。第一次运行后才能加载参数列表。


允许出现的位置： `pipeline` 块且只能出现一次。


可用的参数类型如下：

- string：字符串类型。
- text：文本类型。
- booleanParam：布尔值类型。
- choice：选项参数类型。
- password：密码类型。



**所有的类型使用例子**：
```groovy
pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: '', description: '输入版本号')

        text(name: 'CHANGE_LOG', defaultValue: '', description: '输入更新日志')

        booleanParam(name: 'IS_DEPLOY', defaultValue: true, description: '是否部署')

        choice(name: 'ENV_TYPE', choices: ['dev', 'uat', 'pre'], description: '选择部署环境')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: '输入密码')
    }
    stages {
        stage('Example') {
            steps {
                echo "版本号： ${params.VERSION}"

                echo "更新日志: ${params.CHANGE_LOG}"

                echo "是否部署: ${params.IS_DEPLOY}"

                echo "部署环境: ${params.ENV_TYPE}"

                echo "密码: ${params.PASSWORD}"
            }
        }
    }
}
```
点击构建后效果如下：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587523888127-5c2cd9f1-db30-406e-bd15-8179c9728b1e.png#align=left&display=inline&height=398&margin=%5Bobject%20Object%5D&name=image.png&originHeight=398&originWidth=924&size=19241&status=done&style=stroke&width=924)


#### 场景一：参数化构建时选择分支


> 需要安装 Git Parameter Plug-In 插件。



```groovy
parameters {
  gitParameter(
      branch: '',
      branchFilter: '.*',
      defaultValue: 'master',
      description: '选择分支',
      name: 'BRANCH_NAME',
      quickFilterEnabled: false,
      selectedValue: 'NONE',
      sortMode: 'NONE',
      tagFilter: '*',
      type: 'PT_BRANCH'
}
```


使用参数话构建方式对于高效率持续构建的场景已经不太合适，我们可以使用 **Jenkins 全局变量**做一些条件判断。


### 4.4 [triggers](https://jenkins.io/doc/book/pipeline/syntax/#triggers)


`triggers` 指令用于定义流水线的触发器。通常，生产环境需要基于手动点击构建进行部署。但在开发环境、测试环境甚至预发布环境，应该对自动构建有更高的集成度，使开发只关注于开发，而不必过多纠结于构建的过程。


允许出现的位置： `pipeline` 块且只能出现一次。


Jenkins 自带的触发器有：

- **cron**：定期触发构建，配置与 Linux 系统一样定时任务管理方式类似，但是还有一些扩展的使用方式，具体参考[ Jenkins cron 语法官方说明](https://jenkins.io/doc/book/pipeline/syntax/#cron-syntax)。
例如： `triggers { cron('H */4 * * 1-5') }`
- **pollSCM**：定期对代码仓库进行检测，如果有变化，则触发构建。
例如： `triggers { pollSCM('H */4 * * 1-5') }`
- **upstream**：由上游任务的执行结果触发。当 B 任务的执行以来 A 任务的执行结果时，A 任务就被称为 B 任务的上游任务。
例如： `triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }
`其中 `threshold` 参数是指上游任务的执行结果是什么值时触发， `hudson.model.Result` 是一个枚举，包括以下值：
   - `ABORTED`：任务被终止。
   - `FAILURE`：构建失败。
   - `SUCCESS`：构建成功。
   - `UNSTABLE`：存在一些错误，但不至于构建失败。
   - `NOT_BUILT`：在多阶段构建时，前面阶段的问题导致后面阶段无法执行。



但是这些触发构建的条件使用场景较少，最常见的场景就是推送代码后触发。


#### 场景一：Gitlab 提交代码自动触发构建任务


> 需要安装 GitLab Plugin 插件。



```groovy
pipeline {
    agent {
        node {
            label 'ansible'
        }
    }

    options {
        gitLabConnection("gitlab")
    }
  
    triggers {
        gitlab(
            triggerOnPush: true,
            triggerOnMergeRequest: true,
            branchFilterType: "All",
            secretToken: "gitlab-tiggers-test"
        )
    }

    stages {
        stage('构建') {
            steps {
                echo "测试 Gitlab tiggers"
            }
        }
    }
 
    post {
        failure {
            updateGitlabCommitStatus name: "build", state: "failed"
        }
        success {
            updateGitlabCommitStatus name: "build", state: "success"
        }
    }
}
```
然后在 Gitlab 配置 Webhook：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587546513161-6e620c0c-e1d8-4364-96d5-ce99f424d788.png#align=left&display=inline&height=128&margin=%5Bobject%20Object%5D&name=image.png&originHeight=128&originWidth=656&size=9689&status=done&style=stroke&width=656)

提交代码后，可自动触发构建。并将构建状态推送至 Gitlab 流水线面板中。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/293995/1587546555813-1fb14231-4eef-4bfa-bbac-c2c995957012.png#align=left&display=inline&height=233&margin=%5Bobject%20Object%5D&name=image.png&originHeight=233&originWidth=1288&size=24576&status=done&style=stroke&width=1288)


#### 场景二：流水线任务过滤分支触发构建


只有 master 分支和 dev 分支提交代码才触发构建任务。
```groovy
pipeline {
    agent {
        node {
            label 'ansible'
        }
    }

    triggers {
        gitlab(
            triggerOnPush: true,
            triggerOnMergeRequest: true,
            branchFilterType: "NameBasedFilter",
            includeBranchesSpec: "master,dev",
            secretToken: "gitlab-tiggers-test"
        )

    stages {
        stage('构建') {
            steps {
                echo "测试 Gitlab tiggers"
            }
        }
    }
}
```
其中用到的参数为：

- **triggerOnPush**：当 Gitlab 触发 Push 事件时，是否执行构建。
- **triggerOnMergeRequest**：当 Gitlab 触发 MergeRequest 事件时，是否执行构建。
- **secretToken**：配置触发器 Token 值。
- **branchFilterType**：只有符合条件的分支才能触发构建，选项有：
   - `All`：所有分支。
   - `NameBasedFilter`：基于分支名过滤，多个分支使用（,）相隔，选项有：
      - `includeBranchesSpec`：配置包括的分支名。
      - `excludeBranchesSpec`：配置排除的分支名。
   - `RegexBasedFilter`：基于正则表达式对分支过滤。选项有：
      - `sourceBranchRegex`：配置正则表达式。



> 此配置无法适用于多分支流水线构建任务。



### 4.5 [stage](https://jenkins.io/doc/book/pipeline/syntax/#stage)


`stage` 指令标识 `stages` 节中的具体阶段。


允许出现的位置： `stages` 节内。


### 4.6 [tools](https://jenkins.io/doc/book/pipeline/syntax/#tools)


`tools` 指令定义流水线中的常用的一些工具。工具在「系统管理」-「全局工具设置」中添加。


允许出现的位置： `pipeline` 块内或 `stage` 指令内。


#### 场景一：使用 maven 工具构建


```groovy
pipeline {
    agent {
        label 'ansible'   
    }
    tools {
        maven 'maven' 
    }
    stages {
        stage('测试') {
            steps {
                sh '/data/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven/bin/mvn --version'
            }
        }
    }
}
```


### 4.7 [input](https://jenkins.io/doc/book/pipeline/syntax/#input)


`input` 指令允许在流水线运行中暂停运行，提示用户输入指定参数。作为用户提交的任何一部分参数都将应用于其他部分。


配置项：

- **message**：必须参数，在用户提交 `input` 时作为说明信息呈现给用户。
- **id**：可选标识符，默认为 `stage` 名称。
- **ok**：自定义确认按钮的文本信息。
- **submitter**：允许提交的用户或组，以逗号分隔的用户列表或外部组名。默认允许任何用户。
- **submitterParameter**：将 `submitter` 的值加入环境变量使用。
- **parameters**：提示提交者一个可选的参数列表，参数列表配置参考 [`parameters`](#M04Q8)。



#### 场景一：最终部署需要用户确认
```groovy
pipeline {
    agent any
    stages {
        stage('部署') {
            input {
                message "是否部署?"
                ok "True"
                submitter "admin"
                submitterParameter "APPROVER"
            }
            steps {
                echo "批准者：${APPROVER}"
                echo "部署环境..."
            }
        }
    }
}
```


#### 场景二：用户选择环境部署
```groovy
pipeline {
    agent any
    stages {
        stage('选择环境') {
            input {
                message "确认部署?"
                ok "确认"
                parameters {
                    choice(name: 'ENV', choices: ['dev', 'uat'], description: '选择部署环境')
                }
            }
            steps {
                echo "部署至 ${ENV} 环境"
            }
        }
    }
}
```


### 4.8 [when](https://jenkins.io/doc/book/pipeline/syntax/#when)


`when` 指令允许流水线根据给定的条件决定是否执行该阶段。至少有一个条件，如果有多个条件，必须全部返回为 `true` 才会执行该阶段。


允许出现的位置： `stage` 指令内。


常用的内置条件：

- **branch**：分支名相同才会执行，只适用于多分支流水线任务。  
例如： `when { branch 'master' }`。
- **environment**：比较环境变量的值相同才会执行。  
例如： `when { environment name: 'DEPLOY_TO', value: 'production' }`。
- **equals**：断言结果为真时才会执行。  
例如：`when { equals expected: 2, actual: currentBuild.number }`。
- **expression**：Groovy 表达式返回的布尔值为真时才会执行。  
例如：`when { expression { return params.DEBUG_BUILD } }`。
- **not**：嵌套条件的布尔值为假时才会执行。  
例如： `when { not { branch 'master' } }`。
- **allOf**：当所有的嵌套条件都为真时才会执行。  
例如：`when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }`。
- **anyOf**：至少有一个嵌套条件为真时才会执行。  
例如： `when { anyOf { branch 'master'; branch 'staging' } }`。
- **triggeredBy**：根据触发器的参数决定是否执行。  
例如：
   - `when { triggeredBy 'SCMTrigger' }`
   - `when { triggeredBy 'TimerTrigger' }`
   - `when { triggeredBy 'UpstreamCause' }`
   - `when { triggeredBy cause: "UserIdCause", detail: "vlinde" }`



#### 场景一：根据分支名部署至不同环境


```groovy
pipeline {
    agent any
    stages {
        stage('deploy to dev') {
            when {
                branch 'dev'
            }
            steps {
                echo "deploy to dev env"
            }
        }
        stage('deploy to prod') {
            when {
                branch 'master'
            }
            steps {
                echo "deploy to prod env"
            }
        }
    }
}
```


#### 场景二：过滤指定分支进行部署


```groovy
pipeline {
    agent any
    stages {
        stage('deploying') {
            when {
                expression { BRANCH_NAME ==~ /(dev|prod)/ }
            }
            steps {
                echo "deploy to ${BRANCH_NAME}"
            }
        }
    }
}
```








