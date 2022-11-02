# jenkins pipline demo




#### pipline demo

- https://jenkins.io/zh/doc/book/pipeline/syntax/
- git 插件 https://jenkins.io/doc/pipeline/steps/git/

```pipline
pipeline{
    agent any
    stages{
        stage(&#34;build&#34;){
            steps{
                echo &#34;11111&#34;
            }
        }
    }
}
```

#### pipeline总体介绍

&gt; **基本结构**

以下每一个部分都是必须的，少一个Jenkins都会报错

```pipline
pipeline{
    agent any
    stages{
        stage(&#34;build&#34;){
            steps{
                echo &#34;hellp&#34;
            }
        }
    }
}
```

- **pipeline** 代表整个流水线，包含整条流水线的逻辑
- **stage** 阶段，代表流水线的阶段，每个阶段都必须有名称。
- **stages** 流水线中多个stage的容器，stages部分至少包含一个stage.
- **steps** 代表stage中的一个活多个具体步骤的容器，steps部分至少包含一个步骤
- **agent** 制定流水线的执行位置，流水线中每个阶段都必须在某个地方执行（master节点/slave节点/物理机/虚拟机/docker容器），agent部分指定具体在哪里执行。`agent { label &#39;***-slave&#39;}`

&gt; **可选步骤**

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
                allure includeProperties: false, jdk: &#39;&#39;,report: &#39;jenkins-allure-report&#39;, results: [[path: &#39;allure-results&#39;]]     
            }
        }
        
        failure {
            script {
                if (gitpuller == &#39;noerr&#39;) {
                    mail to: &#34;${email_list}&#34;,
                            subject: &#34;[jenkins Build Notification] ${JOB_NAME} - Build # ${BUILD_NUMBER} 构建失败&#34;,
                            body: &#34;&#39;${env.JOB_NAME}&#39; (${env.BUILD_NUMBER}) 执行失败\n请及时前往 ${env.BUILD_URL} 进行查看&#34;
                } else {
                    echo &#39;scm pull err ignore send mail&#39;
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

&gt; **变量定义（全局）**

通过def project_name，定义job名称
通过def upstream_list= &#39;****,****&#39; 定义上游job名称，用在触发器里面

&gt; **options**

用于配置整个pipeline本身的选项

- `buildDiscarder` 保存最近历史构建记录的数量
- `disableConcurrentBuilds` 同一个pipline，Jenkins是默认可以同时执行多次的，此选项是为了禁止pipeline同时执行
- `retry`：当发生失败是进行重试`retry(4)`

    ```
    options {
        buildDiscarder{logRotator(numToKeepStr: &#39;30&#39;)} # 保存最近x个job执行记录
        timeout(time:1, unit: &#39;HOURS&#39;) # 1小时内未执行完，自动结束
        disableConcurrentBuilds() # 不允许两个job同时执行
    }
    ```
&gt; **parameters**

该parameters指令提供用户在触发Pipeline时应提供的参数列表。这些用户指定的参数的值通过该params对象可用于Pipeline步骤

字符串类型的参数，例如：`parameters { string(name: &#39;DEPLOY_ENV&#39;, defaultValue: &#39;staging&#39;, description: &#39;&#39;) }`

booleanParam，一个布尔参数，例如：`booleanParam(name: &#39;DEPLOY_BUILD&#39;, defaultValue: true, description: &#39;&#39;) }`

目前支持`[booleanParam, choice, credentials ,file, test, password, run, string]`

```
parameters {
    choice(name: &#39;environ&#39;,choices: &#39;test\ndev\nstg&#39;, description: &#39;测试环境，请选择dev? test? stg? &#39;)
    string(name: &#39;keywords&#39;, defaultValue: &#39;&#39;, description: &#39;测试用例名的关键字，用于过滤测试用例&#39;)
    string(name: &#39;folder&#39;, defaultValue: &#39;&#39;, description: &#39;文件夹名称，用于指定具体包那个文件夹下的case&#39;)
}
```

&gt; **triggers配置**

triggers指定定义了流水线被重新出发的自动化方法。当前可用的触发器是cron, pollSCM, upstream, gitlab。例如

```
triggers {
    poolSCM(&#39;H * * * 1-5&#39;) // 周一到周五，每小时
    cron(&#39;H H * * *&#39;) // 每天
    gitlab(triggerOnPush: true, triggerOnMergeRequest: false, branchFilterType: &#39;All&#39;)
    upstream(upstreamPorjects: &#34;${upstream_list}&#34;, threshold: hudson.model.Result.SUCCESS)
}
```

&gt; **定时触发：cron**

接收cron样式的字符串来定义要重新出发流水线的常规间隔，比如`cron(&#39;H H * * *&#39;)` ，每天轮询代码仓库：pollSCM

接收cron样式的字符串来定义一个固定的间隔，在这个间隔中，Jenkins会检查新的源代码更新。如果存在更改，流水线就会被重新出发。例如pollSCM(&#39;H * * * 1-5&#39;) 周一到周五，每小时

由上游任务触发：upstream

接受逗号分割的工作字符串和阈值，当字符串中的任何作业以最小阈值结束时，流水线被重新触发。例如：`triggers { upstream(upstreamPorjects: &#34;job1,job2&#34;, threshold: hudson.model.Result.SUCCESS) }`

hudson.model.Result包括以下状态：
- `ABORTED`：任务被手动终止，
- `FAILURE`：构建失败
- `SUCCESS`：构建成功
- `UNSTABLE`：存在一些错误，但构建没失败
- `NOT_BUILT`：多阶段构建时，前面阶段问题导致后面阶段无法执行

由gitlab触发：gitlab `gitlab(triggerOnPush: true, triggerOnMergeRequest: false, branchFilterType: &#39;All&#39;)`，更改后push到远端。

- `triggerOnPush: true` 代表有push就会触发job
- `triggerOnMergeRequest: false` 代码有merge不会触发
- `branchFilterType: &#39;All&#39;` 所有分支均会触发 

https://blog.csdn.net/qq_30758629/article/details/93353437

***
```pipline
environment {
    git_url = &#39;https://gitlab.com/lc.chow/jenkins-test.git&#39;
    git_key = &#39;176b96d4-0865-4cb8-871d-f9b65a84cecc&#39;
    git_branch = &#39;master&#39;
    gitpullerr = &#39;noerr&#39;
    email_list = &#39;test.com@gmail.com&#39;
}

options {
    buildDiscarder{logRotator(numToKeepStr: &#39;30&#39;)} # 保存最近x个job执行记录
    timeout(time:1, unit: &#39;HOURS&#39;) # 1小时内未执行完，自动结束
    disableConcurrentBuilds() # 不允许两个job同时执行
}

parameters {
    choice(name: &#39;environ&#39;,choices: &#39;test\ndev\nstg&#39;, description: &#39;测试环境，请选择dev? test? stg? &#39;)
    string(name: &#39;keywords&#39;, defaultValue: &#39;&#39;, description: &#39;测试用例名的关键字，用于过滤测试用例&#39;)
    string(name: &#39;folder&#39;, defaultValue: &#39;&#39;, description: &#39;文件夹名称，用于指定具体包那个文件夹下的case&#39;)
}

stags {
    stage(&#39;拉去测试代码&#39;) {
        steps {
            git branch: &#34;${git_branch}&#34;, credentialsId: &#34;$git_key&#34;, url: &#34;$git_url&#34;
        }
    }
    
    stage(&#39;安装测试依赖&#39;) {
        steps {
            sh &#34;pipenv --rm&#34;
            sh &#34;pipenv install --skip-lock --ignore-pipfile&#34;
            sh &#34;pipenv graph&#34;
        }
    }
    
    stage(&#39;执行测试用例&#39;) {
        steps {
            sh &#34;rm -fr $env.WORKSPACE/allure-*&#34;
            sh &#34;pipenv run py.test --env &#39;${params.environ}&#39; -k &#39;${params.keywords}&#39; tests/{params.folder}&#34;
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
        stage(&#39;Depoly pay13&#39;){
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: &#39;52.229.166.83 pay13&#39;, transfers: [sshTransfer(cleanRemote: false, excludes: &#39;&#39;, execCommand: &#34;cd /data/paysCenter &amp;&amp; sudo git pull &amp;&amp; sudo chown -R nginx.nginx /data/paysCenter&#34;, execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: &#39;[, ]&#43;&#39;, remoteDirectory: &#39;&#39;, remoteDirectorySDF: false, removePrefix: &#39;&#39;, sourceFiles: &#39;&#39;)], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])   
                sshPublisher(publishers: [sshPublisherDesc(configName: &#39;52.229.166.83 pay13&#39;, transfers: [sshTransfer(cleanRemote: false, excludes: &#39;&#39;, execCommand: &#39;cd /data/paysCenter &amp;&amp; sudo git pull &amp;&amp; echo `git show|head -1|awk \&#39;{print $2}\&#39;`&#39;, execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: &#39;[, ]&#43;&#39;, remoteDirectory: &#39;&#39;, remoteDirectorySDF: false, removePrefix: &#39;&#39;, sourceFiles: &#39;&#39;)], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
    
    post {
        success {
            sh &#39;&#39;&#39;
            echo &#34;push success&#34;
            &#39;&#39;&#39;
        }
        
        failure {
            sh &#39;&#39;&#39;
                echo &#34;push fail&#34;
            &#39;&#39;&#39;
        }
        
    }
}
```


```pipline
pipeline{
    agent any
    
    environment {
        host = &#34;192.168.50.32&#34;
        path1 = &#34;/data/yabo_appdown&#34;
    }
    
    options {
        timeout(time: 1, unit: &#39;HOURS&#39;)
        buildDiscarder(logRotator(numToKeepStr: &#39;100&#39;))
        disableConcurrentBuilds()
    }
    
    stages{
        
        stage(&#34;pull&#34;){
            steps{
                git branch: &#39;pre&#39;, credentialsId: &#39;abe7e165-b646-472e-ab48-024004ecc589&#39;, url: &#39;http://j7.hnxmny.com:8088/a/front/yabo_appdown&#39;
            }
        }
        
        stage(&#34;build&#34;){
            steps{
                withEnv([&#39;PATH&#43;EXTRA=/usr/sbin:/usr/bin:/sbin:/bin&#39;]) {
                    sh &#34;sudo npm i &amp;&amp; sudo npm run isol&#34;
                }
                
            }
        }
        
        stage(&#34;deploy&#34;){
            steps{
                sh &#39;&#39;&#39;
                    file=`date &#43;%Y%m%d%H%M%S`
                    sudo mv dist dist.${file}
                    tar zcf dist.${file}.tar.gz dist.${file}
                    
                    ansible ${host} -b -m copy -a &#34;src=dist.${file}.tar.gz dest=/tmp/&#34;
                    ansible ${host} -b -m raw -a &#34;tar xf /tmp/dist.${file}.tar.gz -C ${path1}/ &amp;&amp; rm -f /tmp/dist.${file}.tar.gz&#34;
                    ansible ${host} -b -m raw -a &#34;ln -svnf ${path1}/dist.${file} ${path1}/dist&#34;
                &#39;&#39;&#39;
            }
        }
    }
    
    post {
        cleanup {
            echo &#39;One way or another, I have finished&#39;
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
        host = &#34;192.168.50.32&#34;
        path1 = &#34;/data/yabo_appdown&#34;
    }
```

解决方法：

- https://stackoverflow.com/questions/43987005/jenkins-does-not-recognize-command-sh

withEnv 

```
node {
  stage (&#39;STAGE NAME&#39;) {
    withEnv([&#39;PATH&#43;EXTRA=/usr/sbin:/usr/bin:/sbin:/bin&#39;]) {
      sh &#39;//code block&#39;
    }
  }
}
```

- **避免使用系统环境变量名称**


&gt; **进入某个路径下操作**

执行完后会退回原目录

```
dir(path: &#39;ssr-server&#39;) {
    sh &#39;&#39;&#39;2
        rm -fr  node_modules &amp;&amp; npm install
        npm run prestart:prod
    &#39;&#39;&#39;
}
```

&gt; **使用参数**

```pipeline
parameters {
	choice(name: &#39;vernum&#39;,choices: &#39;v1\nv2&#39;, description: &#39;请选择v1? v2?&#39;)
}
```

&gt; **if**

```pipeline
stage(&#34;build&#34;){
	steps{
		script {
			if ( params.vernum == &#39;v1&#39; ) {
				dir(path: &#39;v1&#39;) {
                    sh &#39;&#39;&#39;
                    	npm i
                    &#39;&#39;&#39;
				}
			} else {
				dir(path: &#39;v2&#39;) {
                    sh &#39;&#39;&#39;
                    npm i
                    &#39;&#39;&#39;
				}
			}
		}
	}
}
```


