# jenkins检查代码 如没更新停止构建步骤




### 需求分析

在jenkins中没有找到构建前插件，每次构建时间很长，希望可以实现判断代码是否更新，如果没更细则停止构建步骤。 

### 实现步骤

在构建时执行shell命令，而jenkins提供的的环境变量可以实现此判断 https://wiki.jenkins.io/display/JENKINS/Conditional+BuildStep+Plugin

```
GIT_COMMIT
    The commit hash being checked out.
GIT_PREVIOUS_COMMIT
    The hash of the commit last built on this branch, if any.
GIT_PREVIOUS_SUCCESSFUL_COMMIT
    The hash of the commit last successfully built on this branch, if any.
```

`GIT_COMMIT` 当前拉取版本的commit id
`GIT_PREVIOUS_COMMIT`  最后在此分支上构建的 commit id
`GIT_PREVIOUS_SUCCESSFUL_COMMIT` 最后在此分支上成功构建的 commit id号

```
#!/bin/bash

if [ $GIT_PREVIOUS_SUCCESSFUL_COMMIT == $GIT_COMMIT ];then
　　echo "no change，skip build"
　　exit 0
else
　　echo "git pull commmit id not equals to current commit id trigger build"
fi
```

注意，不能使用-eq 只能使用==
![](https://img2020.cnblogs.com/blog/1380340/202108/1380340-20210801143636307-915398192.png)

提交新版本后，构建提示如下：

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801143624033-1191327542.png)

```
$ git show|head -1
commit 27617e680d2e6bf00062700623792aef63926edd
```

在jenkins中执行构建
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801143657919-271057259.png)

 
