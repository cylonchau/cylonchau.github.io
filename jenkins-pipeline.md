# jenkins pipline demo




#### pipline demo

- https://jenkins.io/zh/doc/book/pipeline/syntax/
- git 插件 https://jenkins.io/doc/pipeline/steps/git/

```pipline
pipeline{
    agent any
    stages{
        stage("build"){
            steps{
                echo "11111"
            }
        }
    }
}
```

#### pipeline总体介绍

> **基本结构**

以下每一个部分都是必须的，少一个Jenkins都会报错

```pipline
pipeline{
    agent any
    stages{
        stage("build"){
            steps{
                echo "hellp"
            }
        }
    }
}
```

- **pipeline** 代表整个流水线，包含整条流水线的逻辑
- **stage** 阶段，代表流水线的阶段，每个阶段都必须有名称。
- **stages** 流水线中多个stage的容器，stages部分至少包含一个stage.
- **steps** 代表stage中的一个活多个具体步骤的容器，steps部分至少包含一个步骤
- **agent** 制定流水线的执行位置，流水线中每个阶段都必须在某个地方执行（master节点/slave节点/物理机/虚拟机/docker容器），agent部分指定具体在哪里执行。`agent { label '***-slave'}`

> **可选步骤**

- **post** 包含的是在整个pipeline或stage完成后的附件条件
    - always 论Pipeline运行的完成状态如何都会执行这段代码
    - changes 只有当前Pipeline运行的状态与先前完成的Pipeline的状态不同时，才能触发运行。
    - failure 当前状态为失败时执行
    - success 当前完成状态为成功时执行

- **demo**

    使用`${test}`，可以引入自定义变量

    ```pipeline
    post {
        always {
            script {
                allure includeProperties: false, jdk: '',report: 'jenkins-allure-report', results: [[path: 'allure-results']]     
            }
        }
        
        failure {
            script {
                if (gitpuller == 'noerr') {
                    mail to: "${email_list}",
                            subject: "[jenkins Build Notification] ${JOB_NAME} - Build # ${BUILD_NUMBER} 构建失败",
                            body: "'${env.JOB_NAME}' (${env.BUILD_NUMBER}) 执行失败\n请及时前往 ${env.BUILD_URL} 进行查看"
                } else {
                    echo 'scm pull err ignore send mail'
                }
            }
        }
    }
    ```

pipeline支持的指令

- **`environment`**：用于设置环境变量，可以定义在stage或pipeline部分，环境变量可以向下面的示例设置为全局的，也可以是阶段`stage`级别的。如你所想，阶段`stage`级别的环境变量只能在定义变量的阶段`stage`使用。

- **`tools`**：可定义在pipeline或stage部分，会自动下载并安装我们指定的工具，并将其加入到PATH变量中

- **`input`**：定义在stage部分，会暂停pipeline，提示你输入内容

- **`options`**：用于配置Jenkins pipeline本身的选项，options指令可以定义在stage或pipeline部分

- **`parallel`**：并行执行多个step 

- **`parameters`**：与input不同，parameters时执行pipeline前传入的一些参数

- **`triggers`**：定义执行pipeline的触发器

- **`when`**：当满足when条件时，阶段才会执行
在使用指令时注意每个指令都有自己的作用域，如果指令使用的位置不正确，Jenkins会报错

> **变量定义（全局）**

通过def project_name，定义job名称
通过def upstream_list= '****,****' 定义上游job名称，用在触发器里面

> **options**

用于配置整个pipeline本身的选项

- `buildDiscarder` 保存最近历史构建记录的数量
- `disableConcurrentBuilds` 同一个pipline，Jenkins是默认可以同时执行多次的，此选项是为了禁止pipeline同时执行
- `retry`：当发生失败是进行重试`retry(4)`

    ```
    options {
        buildDiscarder{logRotator(numToKeepStr: '30')} # 保存最近x个job执行记录
        timeout(time:1, unit: 'HOURS') # 1小时内未执行完，自动结束
        disableConcurrentBuilds() # 不允许两个job同时执行
    }
    ```
> **parameters**

该parameters指令提供用户在触发Pipeline时应提供的参数列表。这些用户指定的参数的值通过该params对象可用于Pipeline步骤

字符串类型的参数，例如：`parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }`

booleanParam，一个布尔参数，例如：`booleanParam(name: 'DEPLOY_BUILD', defaultValue: true, description: '') }`

目前支持`[booleanParam, choice, credentials ,file, test, password, run, string]`

```
parameters {
    choice(name: 'environ',choices: 'test\ndev\nstg', description: '测试环境，请选择dev? test? stg? ')
    string(name: 'keywords', defaultValue: '', description: '测试用例名的关键字，用于过滤测试用例')
    string(name: 'folder', defaultValue: '', description: '文件夹名称，用于指定具体包那个文件夹下的case')
}
```

> **triggers配置**

triggers指定定义了流水线被重新出发的自动化方法。当前可用的触发器是cron, pollSCM, upstream, gitlab。例如

```
triggers {
    poolSCM('H * * * 1-5') // 周一到周五，每小时
    cron('H H * * *') // 每天
    gitlab(triggerOnPush: true, triggerOnMergeRequest: false, branchFilterType: 'All')
    upstream(upstreamPorjects: "${upstream_list}", threshold: hudson.model.Result.SUCCESS)
}
```

> **定时触发：cron**

接收cron样式的字符串来定义要重新出发流水线的常规间隔，比如`cron('H H * * *')` ，每天轮询代码仓库：pollSCM

接收cron样式的字符串来定义一个固定的间隔，在这个间隔中，Jenkins会检查新的源代码更新。如果存在更改，流水线就会被重新出发。例如pollSCM('H * * * 1-5') 周一到周五，每小时

由上游任务触发：upstream

接受逗号分割的工作字符串和阈值，当字符串中的任何作业以最小阈值结束时，流水线被重新触发。例如：`triggers { upstream(upstreamPorjects: "job1,job2", threshold: hudson.model.Result.SUCCESS) }`

hudson.model.Result包括以下状态：
- `ABORTED`：任务被手动终止，
- `FAILURE`：构建失败
- `SUCCESS`：构建成功
- `UNSTABLE`：存在一些错误，但构建没失败
- `NOT_BUILT`：多阶段构建时，前面阶段问题导致后面阶段无法执行

由gitlab触发：gitlab `gitlab(triggerOnPush: true, triggerOnMergeRequest: false, branchFilterType: 'All')`，更改后push到远端。

- `triggerOnPush: true` 代表有push就会触发job
- `triggerOnMergeRequest: false` 代码有merge不会触发
- `branchFilterType: 'All'` 所有分支均会触发 

https://blog.csdn.net/qq_30758629/article/details/93353437

***
```pipline
environment {
    git_url = 'https://gitlab.com/lc.chow/jenkins-test.git'
    git_key = '176b96d4-0865-4cb8-871d-f9b65a84cecc'
    git_branch = 'master'
    gitpullerr = 'noerr'
    email_list = 'test.com@gmail.com'
}

options {
    buildDiscarder{logRotator(numToKeepStr: '30')} # 保存最近x个job执行记录
    timeout(time:1, unit: 'HOURS') # 1小时内未执行完，自动结束
    disableConcurrentBuilds() # 不允许两个job同时执行
}

parameters {
    choice(name: 'environ',choices: 'test\ndev\nstg', description: '测试环境，请选择dev? test? stg? ')
    string(name: 'keywords', defaultValue: '', description: '测试用例名的关键字，用于过滤测试用例')
    string(name: 'folder', defaultValue: '', description: '文件夹名称，用于指定具体包那个文件夹下的case')
}

stags {
    stage('拉去测试代码') {
        steps {
            git branch: "${git_branch}", credentialsId: "$git_key", url: "$git_url"
        }
    }
    
    stage('安装测试依赖') {
        steps {
            sh "pipenv --rm"
            sh "pipenv install --skip-lock --ignore-pipfile"
            sh "pipenv graph"
        }
    }
    
    stage('执行测试用例') {
        steps {
            sh "rm -fr $env.WORKSPACE/allure-*"
            sh "pipenv run py.test --env '${params.environ}' -k '${params.keywords}' tests/{params.folder}"
        }
    }
}

post {
    always {
         
    }
}
```


### demo

```
pipeline  {
    agent any
    stages {
        stage('Depoly pay13'){
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: '52.229.166.83 pay13', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "cd /data/paysCenter && sudo git pull && sudo chown -R nginx.nginx /data/paysCenter", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])   
                sshPublisher(publishers: [sshPublisherDesc(configName: '52.229.166.83 pay13', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /data/paysCenter && sudo git pull && echo `git show|head -1|awk \'{print $2}\'`', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
    
    post {
        success {
            sh '''
            echo "push success"
            '''
        }
        
        failure {
            sh '''
                echo "push fail"
            '''
        }
        
    }
}
```


```pipline
pipeline{
    agent any
    
    environment {
        host = "192.168.50.32"
        path1 = "/data/yabo_appdown"
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '100'))
        disableConcurrentBuilds()
    }
    
    stages{
        
        stage("pull"){
            steps{
                git branch: 'pre', credentialsId: 'abe7e165-b646-472e-ab48-024004ecc589', url: 'http://j7.hnxmny.com:8088/a/front/yabo_appdown'
            }
        }
        
        stage("build"){
            steps{
                withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']) {
                    sh "sudo npm i && sudo npm run isol"
                }
                
            }
        }
        
        stage("deploy"){
            steps{
                sh '''
                    file=`date +%Y%m%d%H%M%S`
                    sudo mv dist dist.${file}
                    tar zcf dist.${file}.tar.gz dist.${file}
                    
                    ansible ${host} -b -m copy -a "src=dist.${file}.tar.gz dest=/tmp/"
                    ansible ${host} -b -m raw -a "tar xf /tmp/dist.${file}.tar.gz -C ${path1}/ && rm -f /tmp/dist.${file}.tar.gz"
                    ansible ${host} -b -m raw -a "ln -svnf ${path1}/dist.${file} ${path1}/dist"
                '''
            }
        }
    }
    
    post {
        cleanup {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
    }
}
```

pipline 报错

```
nohup: failed to run command ‘sh’: No such file or directory
Sending interrupt signal to process
```

原因：`environment`设置了`path`环境变量导致重写出现的原因

```
    environment {
        host = "192.168.50.32"
        path1 = "/data/yabo_appdown"
    }
```

解决方法：

- https://stackoverflow.com/questions/43987005/jenkins-does-not-recognize-command-sh

withEnv 

```
node {
  stage ('STAGE NAME') {
    withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']) {
      sh '//code block'
    }
  }
}
```

- **避免使用系统环境变量名称**


> **进入某个路径下操作**

执行完后会退回原目录

```
dir(path: 'ssr-server') {
    sh '''2
        rm -fr  node_modules && npm install
        npm run prestart:prod
    '''
}
```

> **使用参数**

```pipeline
parameters {
	choice(name: 'vernum',choices: 'v1\nv2', description: '请选择v1? v2?')
}
```

> **if**

```pipeline
stage("build"){
	steps{
		script {
			if ( params.vernum == 'v1' ) {
				dir(path: 'v1') {
                    sh '''
                    	npm i
                    '''
				}
			} else {
				dir(path: 'v2') {
                    sh '''
                    npm i
                    '''
				}
			}
		}
	}
}
```


